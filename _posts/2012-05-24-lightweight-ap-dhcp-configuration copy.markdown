---
layout: post
title: Making IP Helper (UDP Forwarding) HSRP Aware
date: 2011-05-23 11:32:20 +0600
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: network.jpg # Add image post (optional)
tags: [dhcp, ip helper, linkedin, redundancy, tracking]
---

This scenario involves multiple gateway routers using first-hop redundancy protocols like HSRP.  The  routers are using IP helper commands to forward DHCP requests to a central server, however the server is receiving many DHCP requests from the same client, as shown below:

```
dhcpserver#sh log | i through relay
*Mar  1 00:20:41.291: DHCPD: DHCPDISCOVER received from client 0063.6973.636f.2d63 through relay 192.168.1.2
*Mar  1 00:20:43.819: DHCPD: DHCPDISCOVER received from client 0063.6973.636f.2d63 through relay 192.168.1.3
```

It must be understood that DHCP discover packets are broadcast, and thus received and forwarded by all routers configured with IP helper addresses.  To prevent all routers from forwarding the request, the solution is to only allow the active HSRP router to do this.

Below is overall topology:

Configuration for R1:
```
interface FastEthernet0/0
ip address 192.168.1.2 255.255.255.0
ip helper-address 10.1.1.1 redundancy HSRP-Data
duplex auto
speed auto
standby 10 ip 192.168.1.1
standby 10 preempt
standby 10 name HRSP-Data
```

Configuration for R2:
```
interface FastEthernet0/0
ip address 192.168.1.3 255.255.255.0
ip helper-address 10.1.1.1 redundancy HSRP-Data
duplex auto
speed auto
stand 10 ip 192.168.1.1
standby 10 priority 90
standby 10 preempt
standby 10 name HSRP-Data
```

By linking the IP helper command ```ip helper-address 10.1.1.1 redundancy HSRP-Data``` to HSRP it is now able to track the HSRP state.  The IP helper redundancy name must match the HSRP name, in this case ‘HSRP-Data’.   Now when the DHCP client requests an IP configuration, it can be observed that only the active HSRP router replies:

```
dhcpserver#sh log | i through relay
*Mar  1 00:20:41.291: DHCPD: DHCPDISCOVER received from client 0063.6973.636f.2d63 through relay 192.168.1.2
```