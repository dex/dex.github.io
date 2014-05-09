---
layout: post
title: "Open NFLOG socket via tcpdump"
published: true
tags: [Linux, Netfilter, Networking, Ubuntu]
categories: [Linux]
---

Recently, I found that `tcpdump` could be used to receive **NFLOG** packets. But this feature is not out-of-box available for Ubuntu users due to `apparmor` configuration.

In order to receive **NFLOG** packets, you should have the `CAP_NET_ADMIN` capability. By default this is not enabled in `usr.sbin.tcpdump` profile. You have to add the capability to the site-specific profile `/etc/apparmor.d/local/usr.sbin.tcpdump` and reload the profile.

	sudo echo "capability net_admin," >> /etc/apparmor.d/local/usr.sbin.tcpdump
	sudo apparmor_parser -r /etc/apparmor.d/usr.sbin.tcpdump

Now, you can add `iptables` rule to capture packets via `NFLOG`. For example, capture all incoming TCP packets.

	sudo iptables -A INPUT -p tcp -j NFLOG

Then use `nflog` as interface name while using `tcpdump` utility to receive packets:

	sudo tcpdump -i nflog

Furthermore, you seperated the NFLOG traffic into differnt groups, use `nflog:<group_id>` as the interface name.


Reference: [Apparmor](https://help.ubuntu.com/community/AppArmor)
