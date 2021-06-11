---
layout: post
title: Linux BIND Nameserver Delegation for Isilon Smart Connect
permalink: bind-dns-isilon.html
description: Some Description
date: 2021-06-11 9:00:00
tags: linux 
published: true
---
Isilon Smart Connect is a feature that automatically connects users to their access zones based on the DNS name they connect to.  For example, if you have are using DNS zone vdude.local, you can have tenant1.isilon.vdude.local and tenant2.isilon.vdude.local point connect to 2 separate access zones on the Isilon.  However, to do this you need to delegate the Isilon as a nameserver in your domain/DNS zone.  

So if your domain is vdude.local, and you want your storage to be X.isilon.vdude.local, the Isilon itself becomes the dns server of isilon.vdude.local.  You create the X.isilon.vdude.local address in the isilon when you create a new network pool.

Hopefully that all makes sense.  As a Windows guy in a Linux world, it was a little confusing at first.  

Using active directory, configuring the nameserver delegation is pretty easy and there are a lot of guides for that.  But using BIND on linux proved to be kind of tricky, but ultimately simple.

In your zone file, you must add an A record for the isilon, and then an NS record for the isilon.vdude.local subdomain, like so:

```bash
...Zone vdude.local...
smart-connect.vdude.local   IN  A 192.168.1.1
...
...Other IP A records in your zone...
...
$ORIGIN isilon.vdude.local.
@       IN      NS      smart-connect.vdude.local.
```
Next, and don't miss this step because I did and it was very frustrating, you must update your named.conf file for that zone and make sure there are explicitly no forwarders.

```bash
zone    "vdude.local" IN {
        type master;
        file    "/var/named/vdude.local.zone";
        forwarders { };
};
```
(Thanks to this post after many hours of searching for pointing this out https://serverfault.com/questions/170231/bind-how-to-delegate-subzone-to-other-dns-server)

Restart the named service, and you should start to resolve smart connect zones that you create.  