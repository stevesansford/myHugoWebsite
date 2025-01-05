+++
date = '2025-01-04T19:35:28-05:00'
draft = false
title = 'Day #14: Fixing a Useless DNS Cache'
tags = ['#100DayaOfCoding']
+++
My plan for today was to create a performance test [module for ZonePy](day-13-meet-zonepy-a-python-powered-dns-server/). I thought it would be neat if there was a module built into the program triggered by a command line flag. When activated, it would run through a list of provided domains and benchmark the time taken to resolve the domain name.

I wasn't too far into the testing process when I realized I made a big mistake with my order of operations in the main program. I kept noticing cached records for root servers and name servers.

>This is rather pointless. Why have a cache if you process every domain name recursively anyway?

Upon further investigation, I discovered that while the cache was being used, it was being used as part of the recursive search. This means ZonePy was decoding the DNS request, finding the domain needed, and then launching into its recursive search process, whereupon it began to check the local cache before making an upstream request.

This is rather pointless. Why have a cache if you process every domain name recursively anyway?

### Order of Operations Matters

The issue is that I need the program to check the cache earlier. As soon as the DNS request is decoded and we know the domain that was asked for, ZonePy needs to check the cache immediately and return any unexpected IP address that it has stored.

I reworked the order of the program's functions. It wasn't a complicated process. I changed the code to include the cache check in the main program loop. 

This is the corrected code:

```
while True:  # listen for incoming requests
    data, addr = sock.recvfrom(512)  # receive DNS request from client
    start_time = time.perf_counter()  # start the clock
    logging.debug(f"DNS request received from {addr[0]}")
    header, question = parse_client_request_packet(data)  # parse client DNS request
    logging.debug(f"Checking cache for {question.name}")
    if cached_record := check_cache(question.name, question.type_):
        response = create_return_packet(header, question, cached_record[0], cached_record[1])
        elapsed_time = round((time.perf_counter() - start_time) * 1000)
        logging.info(f"Cached record found for {question.name} in {elapsed_time}ms")
    else:
        logging.debug(f"No Cached record found for {question.name}")
        resolved_record = resolve(question.name, question.type_)
```
Now, ZonePy will check the local cache as soon as it knows what domain has been requested. The IP address for a cached domain is now returned within 1-3ms. A significant performance improvement already covers the 400-600ms it was taking before, as every call was resolved recursively, with a cache check at each hop. 

>I'll revisit an optimized cache solution in the future.

I'm not sure how I got that mixed up, but the main thing is it's fixed now. I still want to explore other caching options besides a simple text file, but there are many more important tasks that need to be completed that will have a much bigger impact on ZonePy's performance. I'll revisit an optimized cache solution in the future.

### No Need to Reinvent the Wheel

Reinventing the wheel can be entertaining and educational, but I realized, in this case, it's unnecessary. I don't need to finish the perf test function I started. I just need to [download GRC's excellent DNS Benchmark utility](https://www.grc.com/dns/benchmark.htm), and it should give me all the information I need to dial in the performance of ZonePy.

Which I desperately need. Because although it's stable enough to run as [the primary DNS on my MacBook](day-12-an-almost-useable-dns-server/), it's painfully slow, leading to a lot of timeouts. It is not unexpected, considering there's no optimization like asynchronous requests., but I know I can do better.

Fortunately for me, I still have 86 more days to go...