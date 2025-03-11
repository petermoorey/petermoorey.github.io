---
layout: post
title: Verifying DHCP Relay Configuration
date: 2025-03-10 08:38:20 +0000
description: 
img: dhcp.jpg
tags: [dhcp, server, packet capture, wireshark]
---
### Introduction
IT operations teams prepared to change the IP addresses of multiple centralized/global DHCP servers.  This type of change poses a high risk as it depends on the updated DHCP relay configuration being in place on thousands of switches, routers, firewalls and other devices.

To ensure the success of the project I saw an opportunity to systematically verify all settings before the change was scheduled.  This post details the methodology which includes the following high-level steps:

- Gathering data using a packet capture to record all DHCP requests
- Data transformation using Wireshark command-line tools
- Analyzing the data using Python/Excel

### DHCP Basics
Just a quick reminder on the basics of DHCP before we get started...

**DHCP (Dynamic Host Configuration Protocol)**

DHCP is a network management protocol used to automate the process of configuring devices on IP networks. It allows devices to receive IP addresses and other network configuration parameters automatically, without the need for manual intervention. This is essential for managing large networks efficiently. When a device connects to the network, it sends a DHCP request to the DHCP server, which then assigns an IP address and other necessary configuration details to the device.

**DHCP Relays**


A DHCP relay is a network device that forwards DHCP requests from clients to a DHCP server that is not on the same local network. This is useful in large networks where multiple subnets are used, and a centralised DHCP server is preferred. The DHCP relay listens for DHCP requests on its local network and forwards them to the DHCP server. The server processes the request and sends the response back to the relay, which then forwards it to the client. This ensures that devices on different subnets can still receive IP addresses and configuration from a central DHCP server.

### Step 1 - Gathering DHCP Request Data
Before I became involved in the change, the DHCP relay configuration had already been deployed globally.  Despite the servers not being updated yet with the new IP addresses, I knew DHCP relays would still send packets to all configured server IPs, including the new ones.

My approach was to check each DHCP relay IP (representing a subnet/scope) and validate DHCP requests are being sent to both old and new server IPs.

Example DHCP relay configuration, with three DHCP servers.

```cisco
interface GigabitEthernet0/1
 description Interface with IP helper addresses
 ip address 192.168.1.1 255.255.255.0
 ip helper-address 10.0.0.1
 ip helper-address 10.0.0.2
 ip helper-address 10.0.0.3
 no shutdown
```

Using Cisco routers located close to the DHCP servers, I issued the command below to set up a packet capture for UDP packets destined for port 67 (DHCP server) on the specified port-channel interfaces, with a packet length limit of 100 bytes and a buffer size of 100Mb.

```
monitor capture dhcp_capture match ipv4 protocol udp any eq 67 any limit packet-len 100 buffer size 100 interface range port-channel 1.1001 , port-channel 1.1002 both start
```

Once this was complete I exported the files to my laptop for processing.

### Step 2 - Data transformation

I now have multiple packet capture files, one from each of the routers.  To make the rest of the analysis simpler I merged them into a single file using Wireshark command-line tool 'mergecap'.

```bash
/Applications/Wireshark.app/Contents/MacOS/mergecap -w ./dhcp-merged.pcapng *
```

Now I have a single file with all the DHCP requests, I want to extract all possible source and destination IP pairs, so that I can later check the requests are being sent to all servers by the DHCP relays.

I use the Wireshark 'tshark' command-line tool to apply a display filter (selecting only the DHCP servers I'm concerned with), and export the source/destination IPs.  I also deduplicate the conversations and finally save them to a CSV file.


```bash
/Applications/Wireshark.app/Contents/MacOS/tshark -r ./dhcp-merged.pcapng -Y "ip.dst==10.0.0.1 || ip.dst==10.0.0.2 || ip.dst==10.0.0.3" -T fields -e ip.src -e ip.dst -E separator=, | awk '!seen[$0]++' > ../pcap_merged_filtered_deduped.csv
```

1. /Applications/Wireshark.app/Contents/MacOS/tshark: This is the path to the tshark executable, which is a network protocol analyzer.

2. -r ./dhcp-merged.pcapng: Specifies the input pcapng file to read, in this case, dhcp-merged.pcapng.

3. -Y \"(ip.dst\==10.0.0.1 \|| ip.dst\==10.0.0.2 \|| ip.dst\==10.0.0.3)\"\: Sets a display filter to match packets with specific destination IP addresses.

4. -T fields -e ip.src -e ip.dst: Specifies the output format to show only the source and destination IP addresses.

5. -E separator=,: Sets the field separator to a comma.

6. \| awk '!seen[$0]++': Uses awk to remove duplicate lines from the output.

7. \> ../pcap_merged_filtered_deduped.csv: Redirects the output to a CSV file named pcap_merged_filtered_deduped.csv.

Now I have a CSV file with all captured conversations between DHCP relays and DHCP servers.

```bash
cat ./pcap_merged_filtered_deduped.csv
10.122.54.170,10.0.0.1
10.122.54.170,10.0.0.2
10.122.54.170,10.0.0.3
10.205.183.143,10.0.0.3
10.205.183.143,10.0.0.2
10.205.183.143,10.0.0.2
10.205.183.143,10.0.0.1
```

### Step 3 - Analyzing the data

I decided the easiest way to analyze the data was using Python.  I wrote a GenAI prompt to do this...

>Create a Python script that reads a CSV file containing source and destination IP addresses, checks if the destination IPs match a list of configured DHCP servers, and generates an Excel report highlighting any issues. The script should perform DNS lookups for the source IPs and apply conditional formatting to highlight missing requests to DHCP servers. 

It took a few iterations to produce the script below.

```python
import csv
import openpyxl
from openpyxl.styles import PatternFill
import socket

# List of servers to check
servers_to_check = ['10.0.0.1', '10.0.0.2', '10.0.0.3']

# Read the input file and process the data
source_dest_map = {}

with open('pcap_merged_filtered_deduped.csv', 'r') as file:
    reader = csv.reader(file)
    for row in reader:
        source_ip, dest_ip = row[:2]
        if dest_ip in servers_to_check:  # Check if dest_ip is in the list of servers to check
            if source_ip not in source_dest_map:
                source_dest_map[source_ip] = set()
            source_dest_map[source_ip].add(dest_ip)

# Prepare the output data
output_data = []
for source_ip, dest_ips in source_dest_map.items():
    row = [source_ip]
    for server in servers_to_check:
        if server in dest_ips:
            row.append('yes')
        else:
            row.append('no')
    
    # Perform DNS lookup for the source IP
    try:
        dns_result = socket.gethostbyaddr(source_ip)
    except socket.herror:
        dns_result = 'Unknown'
    
    row.append(dns_result)
    output_data.append(row)

# Create an Excel workbook and add a worksheet
wb = openpyxl.Workbook()
ws = wb.active
ws.title = "Analysis"

# Define the headers
header = ['Source IP'] + servers_to_check + ['DNS Result']
ws.append(header)

# Write the output data to the worksheet
for row in output_data:
    ws.append(row)

# Save the workbook
wb.save('output_analysis.xlsx')

print("Analysis complete. The output is saved to dhcp_analysis.xlsx.")
```

The script produces an Excel worksheet with rows for each DHCP relay, and columns for each DHCP server.  The value indicates if DHCP requests are being sent to the server.  If the cell is red it indicates that DHCP relay may not be setup correctly.

![alt text](/assets/img/dhcp_relay_analysis.png)

### Conclusion
Using this process I was able to verify the DHCP relay configuration for over 500 subnets/scopes, prior to any reconfiguration of DHCP services.

Issues were found and safely fixed prior to the major change taking place.
