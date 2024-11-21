+++
date = '2024-11-20T19:01:15-05:00'
draft = false
title = 'Day #13: Meet ZonePy: A Python-Powered DNS Server'
tags = ['#100DayaOfCoding']
+++
Yep. I gave it a name. **ZonePy**. I solved the last issue preventing me from using it as the primary DNS server for my MacBook. After it ran crash-free today for over two hours, I thought it should have a name. ChatGPT suggested a bunch, but ZonePy was my favourite (and the [domain was available](https://zonepy.com)).

There's still a long way to go and much to learn about DNS and Python, but I'm excited that I have a usable server. The issue I sorted out today was a problem where the program would hang waiting for a response from an upstream DNS server.

For example, if a nameserver failed to respond, the program would get stuck. That's why most domains have multiple nameservers for redundancy. One of the limitations of ZonePY is it only processes the first record. It ignores redundancy, but I intend to make that my next project (I think).

## Dealing with an Unresponsive DNS Server
I can't say what's causing the timeout. It could very well be my Starlink ISP. I'm still running a Gen 1 dishy, and it's had some bumps. The motors are broken, and it's stuck in a single orientation. There are good days and bad. I could upload my code to a cloud server (which I may do), and that might resolve the timeouts.

>I'm still running a Gen 1 dishy and it's had some bumps. The motors are broken, and it's stuck in a single orientation.

Regardless, even if that is my issue, non-responsive servers are going to happen, so my program needs to have a mechanism to deal with it. I have two choices: ```socket.settimeout()``` or ```asyncio```. For this quick fix, I went with the timeout option on ```socket```. 

This was the code that caused the hangup. As you can see, there is no mechanism to deal with a lack of response from the ```nameserver```. The program just sits and waits.

```
logging.debug(f"Query sent to {nameserver}")

sock.sendto(query, (nameserver, 53))  # open connection to the nameserver

response, _ = sock.recvfrom(512)
```


I added a one-second timeout. If ZonePy doesn't receive a response from the upstream server in under one second, it returns an ```NXDOMAIN```. Here's the updated code:

```
try:
    logging.debug(f"Query sent to {nameserver}")

    sock.settimeout(1)
    sock.sendto(query, (nameserver, 53))  # open connection to the nameserver

    response, _ = sock.recvfrom(512)
    logging.debug(f"Response received from {nameserver}")
except socket.timeout:
    logging.warning(f"Response from {nameserver} timed out.")
    response = None
finally:
    sock.close()

```

It's a temporary solution, as my long-term plan is to make the resolver fully asynchronous so it can handle a higher volume of DNS queries. It will also asynchronously query all possible nameservers from a given record, which will also take care of an unresponsive server.

This step will probably come after I create a better cache. I'm [looking at Redis](https://www.youtube.com/watch?v=G1rOthIU-uo), and even if it's not the right choice, I'm interested in learning more about how it works. After all, that's [the point of this DNS server project](/day-9-lets-build-a-dns-server). To learn programming and coding.

> I really want to rewrite this server in Haskell and Rust just for fun.

## Running ZonePy as My Primary DNS Server

After this change was implemented, I ran ZonePy on port 53 and pointed my laptop to 127.0.0.1 for its DNS. It ran all afternoon without fault while I did all manner of work tasks. It did feel sluggish at times, but whether that's related to my extremely un-optimized DNS server or my struggling Starlink connection, I do not know.

### Building a Perf-Test Module

Now that the server is functional, I'm going to build a performance test (maybe as a module in the program) so I can test the server against a list of domains and monitor resolution times. It'll be helpful to isolate any potential ISP issues I may be having.

It will also let me quantify changes to the caching, asynchronous operations, hosting environments, code optimizations and probably a few other things. If I can establish a baseline by resolving a list of 1,000 or so domain names, I can get an average resolution time per domain. I don't know if this is how the pros do it, but for my learning exercise, it'll be sufficient for this phase of my program.

## Next Steps for the ZonePy DNS Server

In no particular order: return multiple records, improve the cache, implement asynchronous operations, add proper support for ```IPv6``` and ```HTTPS``` records, refactor and improve the code. I'm sure there are many more things I need. 

Good thing there's still 87 more days to go...


