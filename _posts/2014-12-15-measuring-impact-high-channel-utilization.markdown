---
layout: post
title: Measuring The Impact Of High Wireless Channel Utilisation
date: 2014-12-15 11:32:20 +0600
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: channel-utilization.png # Add image post (optional)
tags: [beacons, channel utilisaton, packet capture, rf, wireless, wireshark ]
---

I don’t know if anyone else has thought of this approach, but I discovered a nice way to assess the impact of a busy RF environment on a wireless stations ability to transmit a frame.

A brief bit of background information- an 802.11 station can only transmit a frame when the channel is not being used.  Unfortunately there can be many reasons why a channel might be busy, for example another station transmitting, or a source of interference like a rogue AP, or a source of noise like a security motion detector.  There are many tools for viewing how busy a channel is, such as examining radio channel utilization reported by APs, spectrum analysis using Cisco CleanAir or Metageek Channelyzer Pro, these provide good visualisations of utilisation.

Here’s the problem though- they don’t tell you the impact to a client station- how is the interference affecting a station trying to get air time to transmit its frame. Does it have to wait 100ms to send a frame, or 800ms, if the station is sending packets from a real-time application like Microsoft Lync, or Cisco IPT for example, waiting 800ms to get air time is not acceptable.

Here’s an example of a radio that is busy, as you can see the source of the high utilisation is interference, in this case due to a large number of rogue APs sending beacons at 1Mbit/sec using inefficient CCK modulation.

<img src="https://pmoorey.files.wordpress.com/2014/12/channel-util.png?w=600&h=408">

So, how are wireless stations being impacted by this?  Using a wireless packet sniffer we can capture the beacons for the SSID that we want to analyse.  Beacons are sent every 100ms by default, for each SSID.  By looking at the delta time between the transmission of the beacons we can understand how long the AP needs to wait before getting air time.

Here’s a Wireshark display filter to capture beacons for a specific BSSID-

‘wlan.fc.type_subtype == 0x0008 && wlan.bssid == xx:xx:xx:xx:xx:xx’

Using the WireShark I/O graph we can see in the example below that the average delta between beacons is 100ms, the AP is able to get air time very easily.

<img src="https://pmoorey.files.wordpress.com/2014/12/good1.png?w=1088">

Here’s another example where the RF is busy, there are very significant delays in sending beacons.  At times there is a three second delay between transmission of beacons, it’s not hard to imagine the impact of this type of delay on an interactive application like voice, video or desktop sharing.

<img src="https://pmoorey.files.wordpress.com/2014/12/bad.png?w=1088">

There are some caveats to this approach, for example beacons may be given lower priority to other packets, such as those marked with high priority 802.11e values.  There may also be other factors that cause delay in transmission of beacons, such as utilisation of resources (CPU etc.) on the AP.  It’s not a fool proof approach, but I find it is a fairly accurate way to understand the impact of high channel utilisation on a clients ability to transmit.

Another approach could be to use an IP SLA to measure deviation in latency to a client, but this may also measure other components of the network in the path to the destination.