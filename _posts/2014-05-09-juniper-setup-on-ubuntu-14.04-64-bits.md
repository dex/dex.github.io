---
layout: post
title: "Juniper setup on ubuntu 14.04 64-bits"
published: true
tags: [Juniper, Networking, Ubuntu, Linux]
categories: [Linux]
---

The Juniper NetConnect only works with 32-bits JRE. In order to load NetConnect in 64-bits platform you have install 32-bits JRE.

	sudo apt-get install openjdk-7-jre icedtea-7-plugin
	sudo apt-get install openjdk-7-jre:i386
	sudo ln -s /usr/bin/update-alternatives /usr/sbin/

Here is the reference at this topic: [Juniper setup on 12.04](http://askubuntu.com/questions/136194/juniper-setup-on-12-04)
