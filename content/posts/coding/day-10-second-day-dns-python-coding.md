+++
date = '2024-11-17T21:51:51-05:00'
draft = false
title = 'Day #10: My Second Day of DNS Coding in Python'
tags = ['#100DaysOfCoding']
+++
From my last post, you'll recall I copy-coded two variations on [a Python DNS server/resolver](day-9-lets-build-a-dns-server.md). The first project was a server that functions similarly to a nameserver, returning results from a local zone file in response to a request from a client. 

The second was a [custom DNS resolver written in Python](https://implement-dns.wizardzines.com/) that took a domain name input and recursively searched through the Internet DNS hierarchy until it returned an IP address. 

As the second project is closer to what I had in mind for my own DNS server project, I wanted to start working through the suggested exercises to expand the functionality of my resolver.

## Exercise #1: Resolve CNAME Records

Attempting to resolve CNAME records caused the program to crash. This was evident by attempting to resolve ```www.facebook.com``` which returned a ```CNAME``` record pointing to ```facebook.com``` rather than an ```A``` record containing an IP address.

Here's a code snippet from the original code that parsed the response:

```
TYPE_A = 1
TYPE_NS = 2
#...
def parse_record(reader):
    #...
    if type_ == TYPE_NS:
        data = decode_name(reader)
    elif type_ == TYPE_A:
        data = ip_to_string(reader.read(data_len))
    else:
        data = reader.read(data_len)
    return DNSRecord(name, type_, class_, ttl, data)
```
As you can see, it only supports ```A``` and ```NS``` records. A ```CNAME``` record wouldn't be captured and further decoded. Here's my implementation:

```
TYPE = ('', 'A', 'NS', 'MD', 'MF', 'CNAME', 'SOA', 'MB', 'MG',
        'MR', 'NULL', 'WKS', 'PTR', 'HINFO', 'MINFO', 'MX', 'TXT')
#...
def parse_record(reader):
    #...
    if type_ == TYPE.index('NS'):
        data = decode_name(reader)
    elif type_ == TYPE.index('CNAME'):
        data = decode_name(reader)
    elif type_ == TYPE.index('A'):
        data = ip_to_string(reader.read(data_length))
    else:
        data = reader.read(data_length)
```
You can see two changes. Adding an ```elif``` to process the CNAME response allows the resolver to continue towards finding the correct ```A``` record instead of erroring out. As ```NS``` and ```CNAME``` call the same function, I may combine those into a single ```or``` condition. This ```if``` statement will get longer as I add more record types.

I also removed the variables for each record type and replaced them with a tuple containing all the possible record types (as of RFC 1035). This makes it simpler, as I don't need to worry about which integer corresponds with which record type. I just need to call my ```type``` tuple with the ``` index()``` method and the name of the record. The integer is returned.

Exercise #1 is complete. My resolver can now handle CNAME resolution. I think this may be the easiest of the exercises to implement.

## Exercise #4: Add Caching to the DNS Resolver

While there are many more record types to add, I wanted to move on to caching. This is an important part of a DNS resolver. As the resolver goes about turning domain names into matching IP addresses, it should also keep a record of those matches in local storage. 

This allows the DNS server to instantly respond to a query from a client without having to perform a full recursive search. This speeds up the response time and greatly reduces the amount of DNS traffic on the network.

I had a process worked out in my head that went something like this:

1. Check the local cache for a domain name.
2. If found, return the matching IP address.
3. If not, run a recursive search.
4. Write the resulting IP to the cache.
5. Return the IP to the user.

It seemed pretty simple, but then I got lost down the rabbit hole of trying to figure out the best data structure to use for a cache. I'm leaning toward something like Redis, but for now, I need a cache that works. I can optimize it later. I'm going to use a text file. 

### Step One: We Need a Cache to Check

Assume the first DNS query won't have a cached entry, let's use that data to start the process. This first function uses ```pickle``` to serialize and encode the DNS record and write it to the cache file:

```
def cache_dns_record(dns_record):
    serialized = base64.b64encode(pickle.dumps(dns_record)).decode('utf-8')
    with open('cache.txt', 'a') as file:
        file.write(serialized + "\n")
```
Every time we use a recursive search to find a record, we write that record to the cache, along with a timestamp, which we'll use later to determine TTL. 

I'll need to implement a future method to purge records with expired TTLs from the cache. No point in useless records taking up resources, but this process can wait until I have a better overall caching solution.

### Step Two: Now We Can Check the Cache

Now, when we attempt to resolve a DNS query, we first check the cache for a record:

```
def check_cache(domain_name, record_type):
    with open('cache.txt', 'r') as file:
        record = {}
        for line in file:
            item = pickle.loads(base64.b64decode(line.strip().encode('utf-8')))
            ttl_expired = True if time.time() - item.timestamp > item.ttl else False
            if item.name == domain_name and record_type == item.type_ and not ttl_expired:
                record['ip_address'] = item.data
                record['ttl'] = item.ttl
                return record
    return 
```
In the reverse of the first function, we open the test file, retrieve the cached records and iterate over them, looking for any that match our domain name and record type. If we find one, we use the timestamp in the record to calculate if the time to live (TTL) has expired.

```
ttl_expired = True if time.time() - item.timestamp > item.ttl else False
```

If the TTL is expired, the cached record is useless, and we proceed to our recursive search to get a fresh record. If the TTL has not expired, we return the cached record, saving us the effort of performing a full search.

### It's Rudimentary, But it Works!

This is far from an optimal Rudimentary implementation of a cache, but for a coding padawan such as myself, it is sufficient for now. I am exploring other data structures for a more optimized cache. Something like Redis that stores in memory sounds like an interesting idea. We'll come back to that one.

## Exercise #7: Create a Full DNS Server
I know I promised you three exercises and so far I've only shown two. I do have the third one complete, but I can't even begin to describe that process here. It deserves its own post. 

>I feel that I'm still just getting started with this DNS server project

It was a 10-hour grind session that involved combining elements from both previous DNS projects, plenty of discovery into byte encoding, DNS packet construction and an eventual "rewrite" of the entire program. If you want to read about it, [check out tomorrow's blog post](day-11-building-python-dns-server/).

And I feel that I'm still just getting started with this DNS server project. I've already decided I want to try re-writing it in another language just for fun. Maybe Rust or Haskell? But that's a project for another day.

Lucky for me, I still have 90 more days to go...
