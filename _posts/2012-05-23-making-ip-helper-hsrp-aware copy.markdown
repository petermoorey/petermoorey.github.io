---
layout: post
title: Lightweight AP – DHCP Configuration
date: 2011-05-24 11:32:20 +0600
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: wireless.jpg # Add image post (optional)
tags: [dhcp, wireless, option 43, option 60, wlc ]
---
Just some findings on lightweight access points, and configuration of DHCP pools.  DHCP is used to tell the LAPs what the wireless controller(s) ap-mgmt address is on boot up.

Option 60 is specified in  a DHCP discover packet by lightweight APs requesting an IP address, it’s used to tell the server what type of device it is, for example a client may set option 60 to ‘Cisco AP c1140’.

The DHCP server can be configured to provide additional options in a DHCP offer packet, only to those client’s that need it.  In this case, if option 60 is ‘Cisco AP c1140’, the server will return option 43, which has the address of the wlc ap-mgmt interface IP addresses in hex format.

```
!
ip dhcp pool ap_manager
   network 128.58.172.128 255.255.255.224
   default-router 128.58.172.129
   dns-server 192.23.23.192 192.23.23.193
   # Identifies type of device
   option 60 ascii "Cisco AP c1140"
   # Specifies IP of WLC controllers 192.168.10.5|192.168.10.20
   option 43 hex f108c0a80a05c0a80a14
   domain-name eur.slb.net
   lease 8
!
```

The hex value is made up as follows:

|f1 = 0xf1 (type)|
|08= 0x8 (total number of octets in both IP addresses 2×4)|
|c0a80a05=192168105 (first IP)|
|c0a80a14=1921681020 (second IP)|

It should be noted that there is a bug on 3750 switches running 12.2(46)SE which prevents option 60 from being specified. “% Invalid input detected at ‘^’ marker.”  is returned because there is a space in the text ‘Cisco AP c1140’.  To resolve this change to a different version of code.