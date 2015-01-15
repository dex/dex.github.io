---
layout: post
title: "IPv6 on Raspberry Pi"
published: true
category: Linux
tags: [Linux, RPi, IPv6, Networking]
---

# Introduction #

There are two types of IPv6 configuration for IPv4.

* **6in4**: This is most common configuration for IPv4 users. This type of IPv6 will use TSP (Tunnel Setup Protocol).
* **6to4**: Automatic tunnel. If you have a public IPv4 address, this method is out-of-box for linux users. It use a mapping between IPv4 address and IPv6 prefix. (e.q. 2002:*XXXX:YYYY*::/48 where *XXXX:YYYY* is IPv4 address in hex). I prefer to use 6to4 tunnel due to native support by linux.

# Configuration #
	
There are two steps to make a IPv6-ready home box.

1. A 6to4 tunnel interface and binding an IPv6 address to it.
2. Setup radvd (optional if you want to setup an IPv6 subnet)

## 6to4 Interface ##

First of all, the `sit` module must be loaded and then the `sit0` interface will be created automatically.

	modprobe ipv6
	modprobe sit

Now we need an  IPv6 address for `sit0` interface according to the IPv4 address of WAN interface.

	ip -6 addr add 2002:XXXX:YYYY::1/16 dev sit0

The prefix must be **16** due to all 6to4 IPv6 addresses should be handled by `sit0`.

Then the default route should be:

	ip -6 ro add 2000::/3 via ::192.88.99.1 dev sit0 metric 1

Or:

	ip -6 ro add 2000::/3 via 2002:c058:6301::1 dev sit0 metric 1

## The radvd daemon ##

If you want to setup an IPv6 subnet, there are two things must be done.

1. Setup routing entry and enable forwarding function
2. Enable radvd daemon

Here we assume the interface name, which is used to send RA, is `eth0`

	ip -6 ro add 2002:XXXX:YYYY:1::/64 dev eth0 metric 1
	sysctl net.ipv6.conf.all.forwarding = 1

In order to send RA, you have to install `radvd` and configure `/etc/radvd.conf` as following

	interface eth0
	{
	   AdvSendAdvert on;
	   prefix 0:0:0:1::/64
	   {
	      Base6to4Interface ppp0;
	   };
	};   

You can replace the `ppp0` with your WAN interface name.
Then run the `radvd` daemon.

# Reference #
* [IPv6 and 6to4 with dynamic WAN IP](http://blog.dev001.net/post/33503557513/dd-wrt-ipv6-and-6to4-with-dynamic-wan-ip)
* [6to4 Tunnel HowTo](http://www.tldp.org/HOWTO/Linux+IPv6-HOWTO/configuring-ipv6to4-tunnels.html)
