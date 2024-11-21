+++
date = '2024-11-19T20:42:12-05:00'
draft = false
title = 'Day #12: An Almost Useable DNS Server'
tags = ['#100DaysOfCoding', 'Python']
+++
I took a much-needed break from [my Python DNS server project](/day-9-lets-build-a-dns-server) yesterday. My coding brain needed a rest after the two-day grind. I also needed time to finish up the last couple of blog posts. That doesn't mean I didn't spend time thinking about the DNS server. I got back to coding again today to add a few new features.

I did add a small bit of code yesterday to provide support for ```SOA``` records. It's not something I would have considered, but interestingly enough, the ```howcode.org``` domain [from the first server tutorial](https://www.youtube.com/watch?v=HdrPWGZ3NRo&list=PLBOh8f9FoHHhvO5e5HF_6mYvtZegobYX2) returns an ```SOA``` record when it's recursively searched. They're handled similarly to ```CNAME```, so it's a quick fix.

## Moving on From print()

Like most beginners, the first built-in function I learned was ```print()```. It's incredibly useful and easy to use, which is why my code to this point is littered with ```print()``` statements. I needed some way to track my way through the program. A lot is going on.

```
import logging

logging.basicConfig(format="{asctime} - {levelname} - {message}", style="{", datefmt="%Y-%m-%d %H:%M", level=logging.INFO)
```

It was time to switch to ```logging```. I added the necessary ```import``` statement, built a basic format string and changed all my ```print()``` statements to some variation of ```logging.x()``` depending on the situation.

I also added a few extra ```DEBUG``` and ```INFO``` lines to get a nice smooth output when running the program.

## Updating the TYPE Tuple for New Record Types
When I added the ```SOA``` record yesterday to the ```TYPE``` tuple, I realized that it worked for the original record types which are indexed from 1 to 16:

```
TYPE = ('', 'A', 'NS', 'MD', 'MF', 'CNAME', 'SOA', 'MB', 'MG',
        'MR', 'NULL', 'WKS', 'PTR', 'HINFO', 'MINFO', 'MX', 'TXT')
```

But now many of those are obsolete, and I was already getting errors from ```AAAA``` and ```HTTPS``` record requests. Their indexes are 28 and 65, respectively. I would end up with a lot of blank elements in my very long tuple and with more than a few useless record types.

```
TYPE = {'A': 1, 'NS': 2, 'CNAME': 5, 'SOA': 6, 'TXT': 16, 'AAAA': 28, 'HTTPS': 65}
```

I figured if I swapped to a dictionary, I could use a ``` key: value``` pair only for the record types that I needed. It's a much cleaner implementation. Now, I can just call the dictionary by the key (```TYPE['CNAME']```), and the correct integer will be returned.

## My Almost Useable Python DNS Server

In the last exercise, Julia Evans suggests trying out [the Python DNS server](https://zonepy.com) by pointing the DNS on your local machine to the program. That sounded like fine. Well aware that things would break, I thought, what better way to test my new DNS server than to start using it to resolve real DNS requests?

I changed the ```SERVER_PORT``` variable to ```53``` (standard DNS port) and adjusted my DNS settings on my MacBook to point to ```127.0.0.1```.

I then spent the next three hours chasing down bugs and refactoring code to smooth out the process. The biggest issue I had was the original code caused the program to process some ```CNAME``` records as ```NS``` records in situations where there is a ```CNAME``` record in the ```answers``` section and an ```NS``` record in the ```authorities``` section. This mostly happened with Apple and Amazon domains. Good thing those aren't important.

```
elif cname := get_cname(response):
    logging.debug(f"Resolving [CNAME] for [{cname}]")
    return resolve(cname, 1)
elif nsIP := get_nameserver_ip(response):
    nameserver = nsIP
elif ns_domain := get_nameserver(response):
    logging.debug(f"Resolving [NS] for [{ns_domain}]")
    nameserver, _ = resolve(ns_domain, 1)
```

The simple fix was to reorder the ``` if/elif``` statement so that ```CNAME``` records were processed first. Full disclosure: this simple fix took me hours to find. That avoided the false positive triggered by the ```NS``` condition. So far, that has resolved the infinite loops I was experiencing.

### Quick Fix (Sort Of) for AAAA and HTTPS Requests

The other issue that broke the code quite often was requests from my MacBook ```IPv6 AAAA``` and ```Type 65 HTTPS``` requests. I now had support in my ```TYPE``` variable for these two record types, but I'm not yet ready to tackle those features. Both of those will be big tasks.

For now, I came up with a shortcut. There are going to be times when the resolver cannot successfully resolve an IP address. Maybe it's invalid. Maybe there's a network issue or some sort of filtering in place that prevents proper resolution. In those cases, a DNS server will return an ```RCODE = 3``` in the header flags, known as an ```NXDOMAIN```. It tells the requester that no records for this domain can be found.

> There are going to be times when the resolver cannot successfully resolve an IP address.

Neither ```AAAA``` nor ```HTTPS``` requests are required in any cases I am aware of. Most computers will request those records, along with a standard ```A``` record. If the ```AAAA``` and/or ```HTTPS``` requests are returned to the client with an ```NXDOMAIN``` the client will rely on the ```A``` record and establish the connection over ```IPv4```. This is a simplification, but it's sufficient for now.

```
def create_nx_domain_packet(dns_header, dns_question):
    header = struct.pack("!HHHHHH", dns_header.id, 0b1000010000000011, 0, 0, 0, 0)
    question = encode_domain_name(
        dns_question.name) + struct.pack("!HH", dns_question.type_, dns_question.class_)

    return header + question
```

I created a function to return a properly formatted DNS response with a ```QD_COUNT``` equal to 0 and a header flag with ```3``` for the ```RCODE```. If the response from the resolver comes back empty, which I've forced in the event of an ```AAAA``` or ```HTTPS``` request, the server returns the ```NXDOMAIN``` packet to the client. This prevents the program from crashing and it continues to accept and process DNS requests.

The DNS server is now able to function as the primary resolver for my MacBook:

```
2024-11-19 20:39 - INFO - Resolved [a.gslb.aaplimg.com] to [17.253.201.8] via Local Cache
2024-11-19 20:39 - INFO - Querying [17.253.201.8] for [kt-prod.v.aaplimg.com]
2024-11-19 20:39 - INFO - Resolved [kt-prod.v.aaplimg.com] to [17.138.175.254] via Recursive Search.
2024-11-19 20:39 - INFO - Resolved [kt-prod.ess.apple.com] to [17.138.175.254] via Recursive Search.
2024-11-19 20:39 - INFO - Request processed in: 433ms
2024-11-19 20:39 - INFO - Querying [198.41.0.4] for [kt-prod.ess.apple.com]
2024-11-19 20:39 - INFO - Querying [192.41.162.30] for [kt-prod.ess.apple.com]
2024-11-19 20:39 - INFO - Querying [17.253.200.1] for [kt-prod.ess.apple.com]
```
### I Also Added a Timer

I wanted to track the time the resolver took to complete the process. I used the ``` time.time_perf()``` function in Python to measure the elapsed time from when the request was received until the moment the response was sent to the client. This will allow me to track the impact on performance as I change and update the code. I wonder if Redis would be faster?

```
start_time = time.perf_counter()
# attempt to resolve the client's request
response = resolve(dns_question.name, dns_question.type_)
# record time
elapsed_time = round((time.perf_counter() - start_time) * 1000)
```

I was also pleased to see that while the recurring searches took 400 to 600ms on average, the local cache returned its records in < 20ms. That's why we cache records.

## One Last Big Hurdle to Overcome

There are probably many more than one, but I am aware of only one right now. If the DNS server sends a query to an upstream server and that connection times out, the server will crash. This is the only bug I know preventing me from using my server as DNS on my MacBook.

I suspect there may be some solution involving asynchronous functions, but tracking down [the exact issue and coming up with a resolution](day-13-meet-zonepy-a-python-powered-dns-server) is a project for another day, which is fine by me.

Because there are still 88 more days to go...
