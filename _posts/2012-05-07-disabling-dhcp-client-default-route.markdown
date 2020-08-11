---
layout: post
title: Disabling DHCP Client Default Route
date: 2011-05-7 11:32:20 +0600
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: network2.jpeg # Add image post (optional)
tags: [default, dhcp]
---

If you do not wish for your router to install the default route it receives when DHCP client is enabled on an interface,  use the command ```ip dhcp client default-router distance 255``` to raise the administrative distance so it’s not entered into RIB.  This is useful if you are using dynamic routing on a WAN interface, and want to use object tracking to detect lack of default routing.

```
!
interface FastEthernet0/0
description WAN LINK
bandwidth 1000000
ip dhcp client default-router distance 255
ip address dhcp
```
