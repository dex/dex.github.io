---
layout: post
title: "IPv6 and DHCPv6-DP configuration for HiNet under Ubuntu 15.04"
published: true
tags: [Linux, IPv6, DHCPv6-DP, HiNet, Ubuntu, PPPoE, Networking]
categories: [Linux]
---

Recently, I registered IPv6 service for my home network from my ISP (i.e., HiNet). HiNet provides at least two ways to get the global IPv6 prefix.

  1. Router Advertisement
  2. DHCPv6 for Prefix Delegation

# Prerequisite #

In order to enable IPv6 in the LAN of my home gateway, the following packages must be installed at least:

  * radvd
  * wide-dhcpv6-client


# PPPoE Configuration #

## Enable IPv6 for PPPD ##

Add the following lines in `/etc/ppp/options`:

	local
	ipv6 ,

## Enable accept_ra for PPPoE ##

Add the following line in `/etc/sysctl.conf`

	net.ipv6.conf.ppp0.accept_ra=2

Set to `2` when IPv6 forwarding is enabled.

## Hook script ##

We need hook script to invoke the wide-dhcpv6-client after IPv6 is ready on PPP interface.

### /etc/ppp/ipv6-up.d/wide-dhcpv6-client ###

	#!/bin/sh
	
	[ -x /etc/wide-dhcpv6/dhcp6c-ifupdown ] || exit 0
	
	IFACE=$1
	MODE=start
	
	export IFACE MODE
	/etc/wide-dhcpv6/dhcp6c-ifupdown

### /etc/ppp/ipv6-down.d/wide-dhcpv6-client ###

	#!/bin/sh
	
	[ -x /etc/wide-dhcpv6/dhcp6c-ifupdown ] || exit 0
	
	IFACE=$1
	MODE=stop
	
	export IFACE MODE
	/etc/wide-dhcpv6/dhcp6c-ifupdown


# DHCPv6-DP Configuration #

## /etc/wide/dhcpv6/dhcp6c.conf ##

	interface ppp0 {
	  #Request Prefix Delegation on ppp0, and give the received prefix id 0
	  send ia-pd 1;
	  script "/etc/wide-dhcpv6/dhcp6c-script-ppp0";
	};
	
	id-assoc pd 1 {
	  prefix-interface br0 {
	    sla-len 0;
	    sla-id 1;
	  };
	};

Since HiNet only provides `/64` prefix for DHCPv6-DP, the `sla-len` must be set to `0`.

## Hook script ##

We need hook script to restart radvd.

### /etc/wide-dhcpv6/dhcp6c-script-ppp0 ###

	#!/bin/sh
	systemctl restart radvd.service

# Router Advertisement Configuration #

## Enable IPv6 forwarding ##

Add the following line in `/etc/sysctl.conf`

	net.ipv6.conf.all.forwarding=1

## /etc/radvd.conf ##

	interface br0
	{
	   AdvSendAdvert on;
	   #AdvLinkMTU 1492;
	   prefix ::/64
	   {
	   };
	   RDNSS 2001:b000:168::1 2001:b000:168::2
	   {
	   };
	};   

