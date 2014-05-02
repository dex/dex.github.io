---
layout: post
title: "IPv6 on Raspberry Pi"
published: true
category: Linux
tags: Linux RPi
---

#Introdouction#

There are two types of IPv6 configuration for IPv4.

*  6in4: This is most common configuration for IPv4 users. This type of IPv6 will use TSP (Tunnel Setup Protocol).
*  6to4: Automatic tunnel. If you have a public IPv4 address, this method is out-of-box for linux users. It use a mapping between IPv4 address and IPv6 prefix. (e.q. 2002:aa:bb::/48 where aa:bb is IPv4 address in hex)

#Configuration#
	
There are three steps to make a IPv6-ready home box.

