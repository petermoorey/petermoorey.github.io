---
layout: post
title: Wireshark Capture Vs Display Filters
date: 2012-11-11 11:32:20 +0600
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: shark.jpg # Add image post (optional)
fig-caption: Image courtesy of https://oceana.org
tags: [berkeley packet filter, bpf, capture, display, filters, packet analysis, software, technology, wireshark]
---

## Background info

Libpcap – API/C/C++ libarary used for packet capture at the link layer on *nix machines
Winpcap – Libpcap API ported to Windows machines for compatibility
Berkeley Packet Filter – format/syntax used for capture filtering withing TCPDump and Wireshark etc
TCP dump – network analyser created by Lawrence Berkeley National Laboratory
Wireshark – network analyser created by Gerald Combs (now Riverbed)

Wireshark uses the Berkeley Packet Filter format for capture filtering, as this is the format used by Libpcap and Winpcap libraries for capturing of packets at the NIC.   It’s generally not possible to use BPF for display filters, however certain filters do overlap.

BPF filter ‘tcp port 25 and host 192.168.1.1’ is a valid capture filter, but will not function as a display filter.
Display filter ‘tcp.port==25 && ip.addr==192.168.1.140’ is the equivalent display filter.

## Capture filter examples

not host 192.168.1.2
tcp port 80
ether host d4:87:d8:14:2f:18

Custom profile capture filters are stored in C:\Users\%username%\AppData\Roaming\Wireshark\profiles\profilename\cfilters

## Display filter examples

!ip.addr == 192.168.1.2 – find all packets where ip.addr is not 192.168.1.2
http.request.uri contains google.com – finds all packets where the URI (uniform resource identifier) contains google.com
eth.src[4:2] == f8:ee  – find f8:ee in field eth.src, start looking from the 4th byte, for the next two bytes

It’s possible to capture packets using tshark (command line) by issuing tshark.exe -R “display filter here”.

Any field within the packet detail can be applied as a filter, for example you can right click on content type field within a HTTP packet and click copy > as filter, as you can apply or prepare as filter.  http.content_type == “image/jpeg”.

A quick way to filter on a specific TCP flow/conversation is to use the TCP stream number, a unique ID assigned by wireshark to each TCP conversation.  The stream ID can be found by examing the TCP header in packet details, field name “tcp.stream”.