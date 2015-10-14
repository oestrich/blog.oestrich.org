---
layout: post
categories:
- linux
- bind
- dns
title: Setting up a Bind Server
---

I've done this a few times before, but everytime I forget how to do it and refinding the information is generally a time consuming process. Below is the bind configuration I have for a local DNS server.


##### /etc/bind/named.conf.local

```bash
zone "example.com" {
  type master;
  file "/etc/bind/db.example.com";
};
```

Make sure the domain inside of the first line `zone "example.com" {` is spell correctly. I spent a good deal of time trying to figure out why a domain was not being loaded and it was just a simple spelling error.

##### /etc/bind/db.starbas.es

```bash
;
; BIND data file for example.com
;
$TTL    604800
@       IN      SOA     ns.example.com. root.example.com. (
                     2015022001         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
        IN      A       192.168.1.2
;
@       IN      NS      ns.example.com.
@       IN      A       192.168.1.2
ns      IN      A       192.168.1.2

host    IN      A       192.168.1.4
alias   IN      CNAME   host.example.com.
```

Make sure the serial number increments each time the file changes. I use the date plus a number "01" that increments. I use two digits because I tend to edit it a lot when first figuring things out.

The line with "SOA" should have your nameserver and the an email address that has a period instead of the "@" sign. The other numbers I took from the example file.
