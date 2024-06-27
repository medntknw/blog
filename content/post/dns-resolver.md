+++
author = "Aniket Patanwal"
title = "How does DNS work?"
date = "2024-03-26"
description = "In this post we will deep dive into what happens behind the scene when you type in a URL in a search bar"
summary = "This post will cover the basics of DNS. It will touch upon on the concepts of DNS resolver and the lifecycle of a searched URL"
+++
## What is DNS?
DNS is Domain Name System which is like a phonebook of the internet. DNS translates domain names (which humans use) to IP address which browsers use.

## What is a DNS resolver?
A DNS resolver is a server which translates the domain names to IP addresses by querying some other server (Root Name Server) or returning result if stored in cache.

## What is a DNS record?
DNS records gives us the information about a domain. It also helps us verify domains which is used in SSL, DMARC etc.\
For example:\
`A` record tells the IP address a domain points to\
`www.google.com A 172.168.1.1`

## What are Hosted Zones?
All DNS records are added in a hosted zone (Route 53, godaddy)\
Lets say you own a domain `eightfold.ai`\
You will create a hosted zone for it in your DNS provider. In the hosted zone you will have multiple records like NS, A, CNAME, TXT
for eightfold.ai and its subdomains.

## What is Authoritative Name Server?
Hosted Zones are served via **Authoritative Name Server**. \
For each domain a NS record is added which tells the Authoritative Name Server its hosted on\
![Example Image](images/ns-record.png)
**PS: Read Note**\
Example: ns1.gns.com, ns-1.awsdns.com

## How To Reach the Authoritative Name Servers to get the IP address of a domain?
To reach the Authoritative Name Server we must have its IP address. \
But we can't just directly query the single name server which holds the records of a domain as it can lead to:\
1. Too much load on the server
2. Too much latency on the request

**Hence its decentralised using DNS Resolver**


## Who is the DNS resolver?
Typically DNS resolving is handled by the ISP.\
The ISP has some servers which handle the network packets and return the IP of the domains requested\
We can also make our own DNS resolver in /etc/resolvd.conf (at machine level)\
If we are using Wifi the router can also act as a DNS resolver.\
![Example Image](/images/dns-base.png)

>Google DNS resolver : 8.8.8.8\
>Cloudflare DNS resolver: 1.1.1.1


**Note: Going forward this we will assume that our router is our DNS resolver**

## What happens if we search for www.google.com?

![Example Image](/images/root-ns.png)

## What are Root Nameservers?
There are total 13 root NS in the world\
a.root-servers.net\
b.root-servers.net\
.\
.\
m.root-servers.net

**This does not mean there are only 13 physical servers. This means there are 13 fixed IP address**

Lets say for Verisign which owns a.root-servers.net\
There are multiple servers around the world but they advertise the same IP address (a.root-servers.net) using **anycast**

### Lets see the root nameservers using below command
```
nslookup -debug -type=ns .
```
Given `.` does not exist in any resolvers cache or in any root NS we can see how the request recursively goes through each root NS if record is not found.

![Example Image](/images/nslookup-debug-root.png)


## What happens after the request reaches the Root NS?
The Root NS then responds with IP of a TLD server which handles the specific TLD (Top Level Domain)\

Here we can see the  TLD servers of `com`\

![Example Image](/images/nslookup-debug-com.png)


The TLD server then responds with the Authoritative NS \

```
nslookup -debug -type=ns eightfold.ai
```
![Example Image](/images/nslookupfinal.png)

The Authoritative NS then holds the records of the domain and returns the IP address.


## Overview
![Example Image](/images/overview.png)

**Note that there is caching at each layer in this flow**

### Checkout this basic DNS resolver I created in C++ to see the above concepts in actions -> [Basic DNS Resolver](https://github.com/medntknw/DNSResolver)


## References:
https://youtu.be/g_gKI2HCElk?si=5g9QVYk1IQkhy07g\
https://www.directedignorance.com/blog/maximum-length-of-domain-name \
https://datatracker.ietf.org/doc/html/rfc1035#section-4.1.1 \
https://www.fir3net.com/Networking/Protocols/dns-nslookup-how-to-find-the-root-servers.html

