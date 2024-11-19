+++
date = '2024-11-18T16:08:52-05:00'
draft = false
title = 'Day #11: Building a DNS Server with Python'
tags = ['#100DaysOfCoding']
+++
This has been an excellent [project for developing my coding skills](day-9-lets-build-a-dns-server.md). So far, I have a working DNS server that listens for incoming DNS requests and returns a response from a local zone file. I also have a DNS resolver that takes a given domain name, does a recursive search through the DNS hierarchy, returns an IP address, and caches the result for future use.

Both are good programs, but I need to combine the resolver and the server program so that instead of looking to a local zone file (like a nameserver), the DNS server will use its resolver to perform a proper search. Then, I'll have a functional recursive DNS server. 

Look out 8.8.8.8!

## Exercise #7: Build a DNS Server

Exercise #7 from the [Implementing DNS in a Weekend](https://implement-dns.wizardzines.com/) webzine series I followed to build the resolver is to add server functionality. Then, instead of using a provided domain name, we can send an actual DNS query to the server and return a properly formatted DNS response to the client. 

It was a grind. At least a ten-hour session, maybe longer. I lost track of the time. But I wasn't able to stop, even though I could have used a break many times. I don't recall the exact steps I followed, but I do recall many of the issues I had to overcome. The whole process culminated in a complete "re-write" near the end.

### Figuring out the Best Place to Start

I have two functioning programs. I needed to pick one to start from and incorporate the elements of the second. Adding additional functionality to the server program seemed like the logical approach. The issue is everything in the server program is encoded manually, byte by byte. 

The resolver script, on the other hand, makes use of ```@dataclasses``` to store our DNS information. It also benefits from ```struct``` functions, which make moving strings to and from bytes much simpler than doing it manually. The functions to process the recursive search are also more complicated than anything in the server program. 

I decided it would be best to begin with the resolver and build the server functionality around it. That may not have been correct.

### Step One: Replacing the Supplied Domain with a DNS Request

It was a simple matter to use ```socket``` to create a listener on ```localhost:5053``` for the DNS server. Then I could ```dig @localhost -p 5053``` and the DNS query would be received by the server.

#### Problem #1: The Client Sends DNS Querys as Byte Strings

It went downhill right from the beginning. The resolver assumes an input in the form of a simple string (```google.com```). Our client is sending a complete query as a byte string:

```
b'K\xdb\x01 \x00\x01\x00\x00\x00\x00\x00\x01\x06google\x03com\x00\x00\x01\x00\x01\x00\x00)\x10\x00\x00\x00\x00\x00\x00\x00'
```
I needed to parse the DNS request from the client to get the information I needed to pass to the resolver function. As the resolver already had a function to parse the response from the upstream nameserver, I thought I'd just use that. 

I'm not sure I've recovered from that particular mistake. Remember [what I said about bad decisions](day-8-is-everything-defined-by-bad-choices)? This started me down the trail of modifying all my perfectly good functions until nothing worked right, and the code base was a mess. But more on that later.

After my total "re-write," I ended up with two similar functions: ```parse_client_response_packet()``` and the original, which I renamed to ```parse_server_response_packet()```. While similar, they process the data in different ways. 

```
def parse_client_request_packet(data):
    reader = BytesIO(data)  # allow us to iterate over the bytestring # read header bytes and unpack
    items = struct.unpack("!HHHHHH", reader.read(12))
    header = DNSHeader(*items)  # build header
    name = decode_name(reader)  # read the requested URL
    data = reader.read(4)  # read the bytes containing the type and class
    
    # format type and class for question
    type_, class_ = struct.unpack("!HH", data)
    
    # build the DNS client question
    question = DNSQuestion(name, type_, class_)

    return header, question
```


You can also see the benefit of the ```struct``` function in Python. It's simple to pass it a format and a byte string and have it return a list of items we can pass to our ```DNSHeader``` object for further processing. We now have populated objects with all the information sent from the client, including the domain name to pass to the resolver.

### Step Two: Resolve the Domain Name

Thanks to our function above, we now have the information we need to submit to our existing resolver function:

```
ip_address, ttl = resolve(dns_question.name, dns_question.type_)
```
It requires two arguments: the domain name and the record type. Both of these are contained in our ```dns_question``` object. The resolver returns the ```ip_address``` and the ```ttl```.

I want to dive further into the functionality of the resolver, but I'm honestly not clear on how it worked before the rewrite. It could benefit from its own post (and maybe it'll get one or seventeen). But I am clear on the problems I had getting it to work.

#### Problems with the Resolver

It was really one problem. I was trying to pass bytes of data to functions that were designed to work with strings. It was a mess. I'd just spent the last hour or so messing them up even more, trying to parse the request byte string. My functions weren't processing anything correctly. 

I also struggled to understand the problem with the byte strings because I couldn't read them. I know the problem stemmed from working with DNS client/server communications in bytes but storing and manipulating the data in my data classes, which use integers and strings. 

All the back-and-forth conversion was a disaster. After four hours of banging my head against the wall, I needed a new plan. And some help.

### Step Three: The Rewrite (and ChatGPT)

I had a sense of the problem. I know it was caused because I approached the integration wrong. I needed to unify the two sections in bytes. That meant I needed a method to convert my data classes into bytes. That would make it easier to populate them with the parsed DNS data and vice versa.

#### Modify the Dataclasses with a __bytes__ method

I had been consulting with ChatGPT while researching data storage methods for my cache. When I asked it how to serialize a custom data class object in Python for file storage, it suggested using ```pickle``` and storing the data as a byte string.

That is fine for file storage, but the problem when I applied the same process in an effort to convert my class objects into byte string was that ```pickle``` also includes the information necessary to deserialize the object, which results in a byte string that is completely useless.

When I asked ChatGPT about this issue, it agreed that ```pickle``` is not the right choice:

> You're correct that pickle is not suitable for encoding data where the exact structure and byte count matter, such as in DNS response encoding. Instead, you can implement a custom __bytes__ method that converts the object to a compact binary format, using Python's struct module or manual encoding.

Ahh...there it was. The solution to the problem. With ChatGPTs help I was able to add custom ```__bytes__``` methods to my ```dataclasses```:

```
@dataclass
class DNSQuestion:
    name: str
    type_: int
    class_: int

    def __bytes__(self):
        name_binary = b''.join(len(label).to_bytes(
            1, 'big') + label.encode('utf-8') for label in self.name.split('.'))
        name_binary += b'\x00'
        return name_binary + struct.pack("!HH", self.type_, self.class_)
```
Now that I had a way to easily convert the ```dataclasses``` to bytes, I could proceed with my rewrite.

#### Starting Back at the Beginning

I had the answers I needed, but I had no idea where to use them. I knew the best thing to do would be to start over. Not from a blank file, but from something just as effective. I commented out every line of code in my script and I started from the beginning.

I don't recall all the changes I made. As I worked my way through the code line by line, I uncommented and updated every line necessary to take advantage of the new classes. 

I only uncommented functions as they were called and never the entire function. Just that part of it that did what I needed. I deleted what seemed like dozens of lines of code. I know several functions were removed completely. 

It took a couple of hours, but starting with a byte string for a ```google.com``` DNS request, I verified the response byte string with ChatGPT at every step. I noted where it said the packet was malformed and corrected it. Line by line, I worked through my script. 

When I was done, the server would accept a DNS request from a client and return an IP address to the console. All that was left now was to encode a proper response and return it to the client.

### Step Four: Building the Response

There's a better way to do this, but after two solid days of coding, I just wanted to get it working. I attempted to create another ```dataclass``` object to contain the response. That would give it a similar structure to the rest of the program, but I couldn't get it to create a properly formed packet. There were errors.

I took a shortcut and created a proper response using ```struct``` and some hard-coded values. This was sufficient to return a proper ```A``` record to the client.

```
def build_response(data):
    dns_header, dns_question = parse_client_request_packet(
        data)  # parse client DNS request
    # attempt to resolve the client's request
    ip_address, ttl = resolve(dns_question.name, dns_question.type_)
    name = dns_question.name
    data = bytes(map(int, ip_address.split('.')))

    # Todo:  refactor and remove hardcoded values
    header = struct.pack("!HHHHHH", dns_header.id, 0x8180, 1, 1, 0, 0)
    question = encode_domain_name(
        name) + struct.pack("!HH", dns_question.type_, dns_question.class_)
    answer = encode_domain_name(
        name) + struct.pack("!HHIH", dns_question.type_, dns_question.class_, ttl, 4) + data

    return header + question + answer    
```
The biggest issue I had was encoding the IP address onto the end of the question string. No matter how I tried, ```dig``` kept complaining that my response stiring was malformed and contained extra characters.

I had to ask ChatGPT for help. it provided this gem that was the key to proper encoding:

```
 data = bytes(map(int, ip_address.split('.')))
```

Everything up to the ```#Todo``` comment is how I want it. The three lines that follow are me building a DNS response using the resolved IP address and some hardcoded integers. I want to rebuild this part of the function to be more dynamic. DNS requests won't always be for a single ```A``` record. There is more work to do.

### Step Five: Resolving a DNS Query

Success at last. I started the server and sent it a query from my terminal using:

```
dig @localhost -p 5053 google.com
```

The response was exactly what I wanted to see:

```
; <<>> DiG 9.10.6 <<>> @localhost -p 5053 google.com
; (2 servers found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 46520
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;google.com.			IN	A

;; ANSWER SECTION:
google.com.		300	IN	A	142.250.191.238

;; Query time: 185 msec
;; SERVER: 127.0.0.1#5053(127.0.0.1)
;; WHEN: Mon Nov 18 19:19:12 EST 2024
;; MSG SIZE  rcvd: 54
```
A perfectly formed DNS response from [my Python DNS server](/portfolio).

There is so much more work to do on this project. Plus, I want to create a series of posts going over the function in detail once it's complete. The problem is it might be one of those programs that is never complete. [Like a chess engine](day-0-a-brief-history-lesson). A perpetual programming project, good for a lifetime of learning or not. Either way,

There are only 89 more days to go...