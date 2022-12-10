---
layout: post
title: My Interview with an AI (ChatGPT)
date: 2022-12-10 18:38:20 +0000
description: 
img: ai.png
tags: [ai, artificial, intelligence, network, automation]
---

ChatGPT by OpenAI became available in November 2022 as a research preview.  It's a chat/dialogue-based AI which is able to converse naturally with startling realism and accuracy. Sign-up for free and try it yourself at https://chat.openai.com/.

I wanted to learn more, and who better to ask about AI than an AI itself!  Please find below my interview with ChatGPT about it's potential use in the domain of computer networking, and automation.

# Interview

The following is an unedited conversation between myself and ChatGPT. 

#### Pete: Hi ChatGPT, please can you give a quick explanation about what AI is?

ChatGPT: Sure, AI stands for artificial intelligence, which refers to the ability of a computer or machine to mimic or simulate human intelligence and behavior. This can include tasks such as problem solving, learning, and decision making. AI can be used in a variety of fields, including healthcare, finance, and robotics.

#### Pete: How could we apply AI to computer networks?

ChatGPT: There are many ways that AI can be applied to computer networks. For example, AI algorithms can be used to optimize network performance by predicting traffic patterns and routing data more efficiently. AI can also be used to improve network security by identifying and blocking potential threats in real time. Additionally, AI can be used to monitor network health and diagnose problems, helping to prevent downtime and improve overall network reliability.

#### Pete: Can AI be used to configure networks?

ChatGPT: Yes, AI can be used to configure networks. In fact, there are already AI-powered tools and systems available that can help automate the process of configuring network settings and parameters. These AI-based tools can use machine learning algorithms to analyze data about the network, such as traffic patterns and usage statistics, to determine the most efficient configuration for the network. By using AI for network configuration, organizations can save time and improve the performance of their networks.

#### Pete: Can AI convert business requirements into a functioning configuration and deploy it to a Cisco router, for example?

ChatGPT: It is possible for AI to convert requirements into a functioning configuration and deploy it to a Cisco router, although this would likely require the development of specialized AI algorithms and software tools. In general, AI can be used to automate many of the tasks involved in configuring and managing network devices, such as routers and switches. This could include analyzing data about the network to determine the optimal configuration, generating the necessary configuration commands, and deploying them to the appropriate devices. However, it is important to note that AI is not a replacement for human expertise and should be used as a tool to assist with network configuration, rather than as a replacement for trained network administrators.

#### Pete: Let's test your ability to create configurations

Pete: Please can you create a configuration for a Cisco router, it should have two WAN connections, one to BT MPLS and the other to AT&T MPLS.  It should also have a Comcast Internet link, and a LAN connection to the core switch.  I'd like the interface descriptions to contain the connection type, provider name and bandwidth; each value should be separated with a dash.

ChatGPT: Sure, here is an example configuration for a Cisco router with the specified connections:

```
!
! Configuration for Cisco router
!

interface GigabitEthernet0/0
 description BT-MPLS-1000Mbps
 ip address 10.0.0.1 255.255.255.0
!
interface GigabitEthernet0/1
 description AT&T-MPLS-500Mbps
 ip address 10.0.0.2 255.255.255.0
!
interface GigabitEthernet0/2
 description Comcast-Internet-100Mbps
 ip address 10.0.0.3 255.255.255.0
!
interface GigabitEthernet0/3
 description LAN-Core-Switch
 ip address 10.0.0.4 255.255.255.0
!
```

ChatGPT: In this configuration, the router has four interfaces: two for the WAN connections to BT and AT&T, one for the Comcast Internet link, and one for the LAN connection to the core switch. The interface descriptions include the connection type, provider name, and bandwidth, with each value separated by a dash.

#### Pete: Cool! That's perfect, please can you shape each MPLS link to 30Mbps and prioritise Microsoft Teams traffic?

ChatGPT: To shape the MPLS links to 30 Mbps and prioritize Microsoft Teams traffic, you can use the following configuration commands:

```
!
! Configuration for shaping and prioritizing traffic
!

class-map match-any MS-Teams
 match protocol ms-teams
!
policy-map Shape-and-Prioritize
 class MS-Teams
  priority percent 30
 class class-default
  shape average 30000000
!
interface GigabitEthernet0/0
 service-policy input Shape-and-Prioritize
!
interface GigabitEthernet0/1
 service-policy input Shape-and-Prioritize
!
```

ChatGPT: In this configuration, we first create a class map called "MS-Teams" that matches traffic for the Microsoft Teams protocol. We then create a policy map called "Shape-and-Prioritize" that includes two classes: one for the MS-Teams traffic, which is given a priority of 30%, and one for all other traffic, which is shaped to an average of 30 Mbps. Finally, we apply the policy map to both WAN interfaces (GigabitEthernet0/0 and GigabitEthernet0/1) using the service-policy command. This will ensure that Microsoft Teams traffic is prioritized on the MPLS links, while all other traffic is shaped to a maximum of 30 Mbps.

#### Pete: Okay, pretty close! Thanks, please can you create a program using Netmiko to deploy the entire configuration to a router?

ChatGPT: Sure, here is an example Python program using the Netmiko library to deploy the entire configuration to a router:

```
# Import necessary modules
from netmiko import ConnectHandler

# Define the router's IP address and credentials
router_ip = "10.0.0.5"
router_username = "admin"
router_password = "password"

# Define the router's device type and the configuration commands to be executed
router_device = {
    "device_type": "cisco_ios",
    "ip": router_ip,
    "username": router_username,
    "password": router_password,
    "secret": router_password,
}

config_commands = [
    "interface GigabitEthernet0/0",
    "description BT-MPLS-1000Mbps",
    "ip address 10.0.0.1 255.255.255.0",
    "interface GigabitEthernet0/1",
    "description AT&T-MPLS-500Mbps",
    "ip address 10.0.0.2 255.255.255.0",
    "interface GigabitEthernet0/2",
    "description Comcast-Internet-100Mbps",
    "ip address 10.0.0.3 255.255.255.0",
    "interface GigabitEthernet0/3",
    "description LAN-Core-Switch",
    "ip address 10.0.0.4 255.255.255.0",
    "class-map match-any MS-Teams",
    "match protocol ms-teams",
    "policy-map Shape-and-Prioritize",
    "class MS-Teams",
    "priority percent 30",
    "class class-default",
    "shape average 30000000",
    "interface GigabitEthernet0/0",
    "service-policy input Shape-and-Prioritize",
    "interface GigabitEthernet0/1",
    "service-policy input Shape-and-Prioritize",
]

# Connect to the router and execute the configuration commands
with ConnectHandler(**router_device) as ssh:
    ssh.enable()
    ssh.send_config_set(config_commands)

# Output a message indicating that the configuration has been applied
print("Configuration applied successfully!")
```

ChatGPT: In this program, we use the Netmiko library to connect to the router and execute the specified configuration commands. The send_config_set method is used to send the configuration commands to the router and apply them. Once the configuration has been applied, a message is output indicating that the process was successful.


#### Pete: I'm curious, please can you show me how to use PyATS to create unit tests which ensure all BGP peers are established, and each peer is receiving at least 100 prefixes?

ChatGPT: Sure, here is an example Python program using PyATS to create unit tests that ensure all BGP peers are established and receiving at least 100 prefixes:

```
# Import necessary modules
from ats.easypy import run
from genie.testbed import load
from genie.libs.sdk.libs.utils.mapping import Mapping
from unittest import TestCase, main

# Define the path to the testbed file and the device under test
TESTBED_FILE = "path/to/testbed.yaml"
DEVICE_UNDER_TEST = "router1"

# Load the testbed and device under test
testbed = load(TESTBED_FILE)
router = testbed.devices[DEVICE_UNDER_TEST]
router.connect()

# Define the class for the unit tests
class BGPUnitTests(TestCase):
    def test_all_peers_established(self):
        # Get the BGP summary information
        bgp_summary = router.parse("show bgp summary")

        # Assert that all BGP peers are in the "Established" state
        self.assertTrue(all(peer["state"] == "Established"
                            for peer in bgp_summary["instance"]["default"]["vrf"]["default"]["neighbor"]))

    def test_each_peer_receiving_100_prefixes(self):
        # Get the BGP summary information
        bgp_summary = router.parse("show bgp summary")

        # Assert that each BGP peer is receiving at least 100 prefixes
        self.assertTrue(all(peer["prefixes"] >= 100
                            for peer in bgp_summary["instance"]["default"]["vrf"]["default"]["neighbor"]))

# Run the unit tests
run(testbed=testbed, tests=BGPUnitTests)
```

ChatGPT: In this program, we use PyATS to load the testbed and connect to the device under test. We then define a class called BGPUnitTests that includes two unit tests: one to ensure that all BGP peers are in the "Established" state, and one to ensure that each peer is receiving at least 100 prefixes. The assertTrue method is used in each test to verify that the specified condition is met. Finally, we use the run method to execute the unit tests on the specified testbed.

#### Pete: Thanks!  Final question - how should people start using AI for network automation?

ChatGPT: To start using AI for network automation, there are a few steps you can follow:

- Identify areas of your network where AI could be beneficial. This could include tasks such as network optimization, security, and monitoring.
- Research and evaluate AI-powered tools and solutions that are available for network automation. Consider factors such as functionality, compatibility with your existing infrastructure, and cost.
- Develop a plan for implementing AI in your network. This should include a timeline, a budget, and a clear understanding of how the AI solution will be integrated with your existing processes and systems.
- Train your team on how to use the AI tools and solutions you have selected. This may involve training on specific AI algorithms and techniques, as well as on the specific tools and platforms you will be using.
- Begin implementing AI in your network, starting with smaller, less critical tasks and gradually expanding to more complex and important tasks. Monitor and evaluate the performance of the AI solutions, and make adjustments as needed.
- Overall, the key to successful implementation of AI for network automation is to start small and focus on specific, well-defined tasks, rather than trying to automate everything at once. This will help you gain experience and confidence with AI, and allow you to gradually expand its use in your network.

#### Pete: Brilliant, thank you!
