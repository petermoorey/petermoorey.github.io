---
layout: post
title: Cisco APIC-EM (SDN Controller) First Impressions
date: 2015-11-24 11:32:20 +0600
description: Cisco APIC-EM (SDN Controller) First Impressions
img: binary.jpg
tags: [cisco, sdn, apic-em]
---

Cisco Application Policy Infrastructure Controller - Enterprise Module (APIC EM) is Cisco's Software Defined Networking (SDN) product for WAN, campus and access networks.  Version 1.0 has recently been released to the public and is available to Cisco DevNet members <a href="https://developer.cisco.com/site/apic-em/" target="_blank">here</a>.  

The concept of APIC EM is that an organization can define and implement network policies without needing to worry about the underlying network hardware or software configuration.  Network automation is enabled through a programmable centralized SDN controller model.  Due to the programmable nature of the controller applications can be written to achieve business goals by interfacing with the controller, and subsequently the network, through a northbound <a href="https://anypoint.mulesoft.com/apiplatform/apx/#/portals/organizations/2f77c6ff-527a-4d99-8548-293066f3028a/apis/41412/versions/42909" target="_blank">application program interface (API)</a> in an 'abstracted' way.

## APIC EM architecture

<img src="https://media-exp1.licdn.com/dms/image/C4E12AQGiuY8EPvlaZg/article-inline_image-shrink_1000_1488/0?e=1602720000&v=beta&t=2v7S2RiGaSO79E1eQxzYnzS5THa4gerQlks3xl9zr1U">

APIC EM is a software based network controller which is included within the Cisco ONE software portfolio, it has no direct cost associated with it which is great news.  The setup process is straightforward.  Using an ISO image a single host VM or bare-metal system can be up and running in around 30 minutes after following a basic installation wizard.  Support for multiple hosts in a scalable and high availability mode is available.  APIC EM uses Grapevine 'elastic services' enabling it to operate using a horizontally scalable Platform as a Service (PaaS) model; each service runs within a separate container enabling it to scale up and down to meet demand.  The Grapevine system is quite interesting in itself and is the first Cisco software product that I've seen using a PaaS approach.

The initial software release supports a wide range of Cisco devices covering routing, switching and wireless product families, it is also surprisingly feature-rich for a first version and includes some built-in applications:

- Cisco iWAN - automated configuration
- Network topology visualization - view network topologies
- Path trace - analysis of end to end application network path
- Plug 'n play - zero touch deployment of network devices

After installing APIC EM the first step is to discover the devices in your network. The controller southbound interface communicates with network devices using SNMP and CLI, other protocols like netconf to be supported in the future.  It builds and maintains a network information database containing the network state, this data can be queried using JSON to get information like link status and other parameters via the APIC EM web service.   I found APIC EM is able to integrate easily into existing networks, quickly and effectively discovering the network topology and devices.  Once you have discovered network devices you can start taking advantage of the controller.  

The built-in application/feature which really interested me the most in the initial release is the Plug n Play server.  This is really useful for enabling zero touch provisioning; device configuration and software images can be pre staged so that when a device is first connected it will 'call home' to the PnP server and securely download its configuration and software image automatically without any manual intervention; this will drastically speed up the configuration process and simplify network operations.  It removes all the hassle of obtaining a console connection, manually copying configurations and upgrading software which can be a very time consuming and error prone process.

## APIC EM Plug 'n Play:

<img src="https://media-exp1.licdn.com/dms/image/C4D12AQEE65nwOXFFJQ/article-inline_image-shrink_1500_2232/0?e=1602720000&v=beta&t=JZX5v7DXUETzYboaQ-G7Fh-xx74coJlnY37ysh0ksWc">

I'm keen to see how Cisco develop the APIC EM controller in the future, the initial release is certainly very promising.  In its current state I don't think there is a huge scope to write customized network policies as the API is fairly limited, no doubt it will be expanded quickly to enable the creation of a broad range of network policies and more network state information will be exposed via the API.

Some of the possible use cases include integration with third party software products (or in-house developed Apps) to dynamically trigger network policies for example enabling QoS, performance based routing (PfR) or network security restrictions etc.  There is a also the potential for communication of network status information to other applications from APIC EM, for example a security product might use APIC EM controller to get the location of a malicious host.  There are certainly a lot of really interesting possibilities!  

To find out more about APIC EM there is a really nice presentation available from Cisco Live 2015 <a href="https://www.ciscolive2015.com/connect/sessionDetail.ww?SESSION_ID=2322" target="_blank">here</a> which I found very helpful.  There is also some excellent information including sandbox demos on the Cisco DevNet website.