+++
date = '2024-11-16T23:16:10-05:00'
draft = false
title = "Day #9: Let's Build a DNS Server!"
tags = ['#100DaysOfCoding']
+++
On Day #2, I posted an article about the [differences between tutorial coding and actual coding](day-2-coding-tutorials-vs-actually-coding/), albeit from my limited perspective. While tutorials are great, it's not until you strike out on your own and start coding a real application that you begin to cement your learning and strengthen your coding muscles.

I also mentioned I'd need to come up with a project suitable for my first program. I looked at all the standard beginner options: a to-do list, a choose-your-own-adventure game, a password generator, etc. All good options, but none felt inspired.

>Let's build a DNS server!

Then I had a thought. Let's build a DNS server. As soon as I had the thought, I knew it was crazy. I have several years of experience working with DNS from my day job. Still, a DNS server is not exactly a beginner project. No way I could code that on my own, but I felt it had a sufficient difficulty level that even if I had to rely on tutorials and sample code, the actual implementation of a DNS server would expand my programming knowledge extensively.

Boy, was I right.

## Step #1: Where Do I Start

I have a solid understanding of DNS from years of experience working with domains and networking. It doesn't begin to prepare me to code my own server. I need some help getting started.

I found a great YouTube series on [building a DNS server with Python](https://www.youtube.com/watch?v=HdrPWGZ3NRo&list=PLBOh8f9FoHHhvO5e5HF_6mYvtZegobYX2). It's over eight years old as of this post, but it's still highly relevant. The first several videos, in particular, where the author explains the RFC 1035 in detail and breaks down the necessary components of a DNS request.

The twist is that DNS queries and replies are always sent in bytes:

```
b'K\xdb\x01 \x00\x01\x00\x00\x00\x00\x00\x01\x06google\x03com\x00\x00\x01\x00\x01\x00\x00)\x10\x00\x00\x00\x00\x00\x00\x00'
```
This is a byte string representing a DNS request for ```google.com```. The lack of human readability makes parsing the question and encoding the reply more difficult because I can't read the data. Fortunately, this tutorial series was quite good and stepped through encoding each element of the string.

As I would learn later, there are more [efficient ways of encoding and decoding byte strings in Python](day-10-my-second-full-day-of-dns-server-coding) (hint: ```struct```), but the manual byte-by-byte encoding of this series helped me form a clear understanding of DNS packet construction.

I won't spoil the entire tutorial, but I got through the entire copy-coding process (with a few of my own minor adjustments) in about four hours. In the end, I had a functional DNS server that would accept a request from a client (via a ```dig @localhost``` command in this case). The program would decode the request, search its local storage for a matching Zone file, encode the DNS response and return the A records to the client.

### A Good Start, But Not What I'm After

This was a great first step. I had a functioning server. The issue was I had built a nameserver. A nameserver is a server that holds the zone file for a particular domain. For example, the zone file for this blog's domain ```stevesansford.com``` resides on a Cloudflare nameserver. 

When that server receives a DNS request for stevesansford.com, the nameserver can access its local records (the zone file) and provide the answer. When your computer first reached out to find the IP address for this blog, my nameserver wasn't the first place it looked. In fact, it was last. 

>Your computer got this blog's IP address from the zone file on a Cloudflare nameserver, which it got from the .com tld server, which it got from a root server, which it found via a recursive DNS server.


What I wanted to build was a recursive DNS server (like Google's 8.8.8.8), the first step in the DNS resolution chain. A recursive DNS server doesn't store Zone files. In fact, outside of recently cached information, a recursive server doesn't store anything. It just knows who to ask when it needs an answer to a DNS query from a client.

I won't expand on the entire process here, but Network Chuck has a [superb primer on how DNS functions](https://www.youtube.com/watch?v=NiQTs9DbtW4). If you want a deeper understanding of what I'm talking about, please go watch his video.

## Step 2: Building a Recursive DNS Resolver
I went back to the Internet in search of the next step. It was difficult. Lots of information on implementing various open-source DNS solutions, like ```bind``` and ```unbound```, but not much on building your own. Then I hit the jackpot!

I found an e-book (webzine?) from developer Julie Evans on her personal blog, titled **[Implement DNS in a Weekend](https://implement-dns.wizardzines.com/)**. It's a step-by-step walkthrough of creating a DNS resolver that takes a provided domain name and recursively searches (```root server > tld server > nameserver```) and returns the IP address for the domain.

The build is broken into three parts, and the author offers four different ways to utilize the project. I went for step 2 with a twist. I didn't copy and paste the code as suggested but copy-coded it into a Python file. I find this process introduces errors in the code that I then have to find and resolve. It's more engaging than copy-and-paste. Debugging is a useful skill for a programmer.

>I learned a lot about the ```@dataclass``` decorator and a couple of clever ways to interact with byte strings. 

Again, I won't spoil the entire project, but once I worked through all three parts, which took me about 8-hours (with no break), I had a fully functional DNS resolver. I learned a lot about the ```@dataclass``` decorator and a couple of clever ways to interact with byte strings. In the end, I could provide the resolver with a domain name, and it would recursively search and return the corresponding IP address.

### Future Exercises for Future Consideration
The webzine also includes seven additional exercises to further [expand the capabilities of the DNS resolver](https://implement-dns.wizardzines.com/book/exercises). I want to do them all, but I'll start with the ones which interest me.

> The three that stood out were adding additional record types, local caching and server functionality.

The first suggestion was to add CNAME support. Exercise 2 adds support for other record types. CNAME lookups would crash the resolver. I'd like to include AAAA (IPv6) and MX records as well.

Caching is another exercise I want to implement. Any good resolver will keep a local cache to reduce network load. I'm not sure of the best data structure to support it, but even adding rudimentary caching with a text file would be better than nothing.

The next one I am going to implement is server functionality. I already copy-coded a functional server, so I plan to integrate the two scripts I have and create a resolver that accepts DNS queries from clients, resolves them with my custom resolver and then returns a properly formatted DNS record to the client.

> Sounds easy, right?

I don't know which other recommended exercises I'll tackle. This project is already big enough for now. Not that I'm worried.

I still have 91 more days to go...

	













