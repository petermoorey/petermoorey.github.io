---
layout: post
title: Get Automating with Cisco Prime Infrastructure API
date: 2015-07-19 11:32:20 +0600
description: Get Automating with Cisco Prime Infrastructure API
img: wifi-symbol.jpg
tags: [cisco, prime, automation, perl]
---

For a Network Engineer the idea of creating a program to interact with an application program interface (API) can seem daunting, but in reality it's not so bad.  Here's a step-by-step guide to using the Cisco Prime Infrastructure (CPI) API to perform the task of adding multiple controllers to Prime, using a mere 16 lines of code...

## Why use an API? 
APIs are really beneficial when interfacing with a system in a programmatic way, for example integrating a process from one system to another.  Automating addition of devices to a monitoring tool may not seem a big win, but in a large environment with hundreds or thousands of devices it can quickly produce some big time/cost savings, and also improve quality by removing opportunity for human error.  There are many other opportunities to use the API beyond this simple example.

## Cisco Prime Infrastructure API
The API documentation is on the CPI server, access it at https://server/webacs/api/v1.  The API supports read (GET) and write (PUT) methods using either XML or JSON format.  Create a user account for performing the API calls, this account should be given appropriate permissions, within the CPI GUI go to Administration > Users, Roles and AAA > Users.

<img src="https://media-exp1.licdn.com/dms/image/C4D12AQFXg5An-rkHqQ/article-inline_image-shrink_1000_1488/0?e=1602720000&v=beta&t=w94bs6urylhIuWmTsAlkH0lM6K8yR9mZucjKOIVeOPo">

In this example we'll be making use of the new device Credential Profiles feature.  This allows us to define a set of credentials (template containing SNMP details) that can be referenced by its name when adding new devices.  To create a profile go to Inventory > Device Management > Credential Profiles and click 'Add'.  

<img src="https://media-exp1.licdn.com/dms/image/C4D12AQEu53JXJK2wVw/article-inline_image-shrink_1000_1488/0?e=1602720000&v=beta&t=CJP6cZOAo_7FljfLsj_Kpov0-moF-_VF5piGX4qKCEA">

## Making API calls
We'll be issuing a HTTP PUT to send a small snippet of XML code containing the DNS name (or IP) of the WLC/s we want to add to CPI.  To issue the HTTP transaction we'll use cURL which is a command line tool for transferring data.  We'll create a small program using Perl to interact with cURL and print some output upon completion.  

## The code!
Let's walk through the main parts of the program (the full program is at the bottom of this post), we start by storing information about the CPI system in variables to be referenced in later in the program:

```perl
my $cpi_server = 'server.domain.com';
my $cpi_user = 'user';
my $cpi_password = 'password';
my $cpi_device_credential_profile = 'my_profile_name';
```

Next we define a list of controllers to add using their DNS names or IPs.  Typically this list is populated by an prior section of code using a database query, or importing a text file for example.  It depends a lot on how you identify the devices to add each time the script runs.

```perl
my @controllers = qw(example-wlc1.domain.com example-wlc2.domain.com);
```

Now we can define the XML code that will be sent to CPI.  Within it are two variables, the name of the controller and the name of the credential profile.

```perl
my $xml = '<?xml version="1.0" encoding="UTF-8" standalone="yes"?><devicesImport><devices><device><credentialProfileName>'.$cpi_device_credential_profile.'</credentialProfileName><ipAddress>'.$controller.'</ipAddress></device></devices></devicesImport>';
```

Next we will define the cURL command (which includes the XML stored in variable $xml).  Also, note the URL contains '/api/v1/op/devices/bulkImport', this is the function of the API on the server that handles importing devices.

```perl
my $api_command = 'curl -s -H "Content-Type: text/xml" -X PUT --data-binary \''.$xml."' -k -u $cpi_user:$cpi_password https://$cpi_server/webacs/api/v1/op/devices/bulkImport";
```

Finally, we can initiate the command to send the XML code to CPI.  At this point the HTTP PUT will be submitted using cURL and the server will create a job which will add the device to CPI.  

```perl
my $api_response = `$api_command`;
```

## The results!
You can view the resulting job/s within the CPI GUI by going to Administration > Jobs.

<img src="https://media-exp1.licdn.com/dms/image/C4D12AQEu53JXJK2wVw/article-inline_image-shrink_1000_1488/0?e=1602720000&v=beta&t=CJP6cZOAo_7FljfLsj_Kpov0-moF-_VF5piGX4qKCEA">

When the job completes you can go to Inventory > Network Devices to see the new controllers which have been added.

<img src="https://media-exp1.licdn.com/dms/image/C4D12AQGgTI2-zi6Yag/article-inline_image-shrink_1000_1488/0?e=1602720000&v=beta&t=eeNSMdRaGHZmyaVSoa9i3UCGS1OxeumrWOLNley9jLY">

The program can be added to crontab (in Linux) to automatically run on a schedule.  Below is an example of the output when running the program.

```
user@linux-server:$ perl ./script.pl
Job_BulkImport_07_19_11_305_PM_07_19_2015 has been created to add controller1.domain.com to server.domain.com
Job_BulkImport_07_19_12_241_PM_07_19_2015 has been created to add controller2.domain.com to server.domain.com
```

## The complete program:
Below is the entire program, it has a couple of extra functions not mentioned above; a loop to add each WLC in the list, and a split function to capture the resulting job ID created by CPI, and print it to the console.

```perl
use strict;
use warnings;
my $cpi_server = 'server.domain.com';
my $cpi_user = 'user';
my $cpi_password = 'password';
my $cpi_device_credential_profile = 'my_profile_name';
my @controllers = qw(example-wlc1.domain.com example-wlc2.domain.com);

foreach my $controller (@controllers){
my $xml = '<?xml version="1.0" encoding="UTF-8" standalone="yes"?><devicesImport><devices><device><credentialProfileName>'.$cpi_device_credential_profile.'</credentialProfileName><ipAddress>'.$controller.'</ipAddress></device></devices></devicesImport>';
my $api_command = 'curl -s -H "Content-Type: text/xml" -X PUT --data-binary \''.$xml."' -k -u $cpi_user:$cpi_password https://$cpi_server/webacs/api/v1/op/devices/bulkImport";
my $api_response = `$api_command`;
my $result_regexp = '<jobName>|</jobName>'; 
my @split_output_xml = split ($result_regexp, $api_response); 
my $job_id = $split_output_xml[1];
print "$job_id has been created to add $controller to $cpi_server\n";
}
```

How are you using the API?  What uses have you found for it?  Drop a note in the comments.
