---
layout: post
title: Top 10 SD-WAN Administrative Security Considerations
date: 2021-09-03 07:38:20 +0000
description: 
img: sd-wan.jpg
tags: [sd-wan, access, control, identity, management, tacacs, saml]
---

I've been working on commercialization of a new SD-WAN solution, including identity and access management for administrative users and system/tools integration.  In this article I'll document some of the key considerations for securing access to any SD-WAN solution.  

Ultimately, the goal is to ensure the security and integrity of the network infrastructure, by identifying risks and establishing controls to reduce the likelihood and severity of CyberSecurity incident.

# 1 - Risk assessment
- Completing a risk assessment before deploying SD-WAN ensures appropriate controls are implemented from the outset.
- Goal of the risk assessment is to identify potential losses/failures, and determine control measures which will reduce/prevent a condition from occurring, or reduce its severity
- Work with your selected vendor to get their input/suggestions
- Review with your CyberSecurity team and other stakeholders

# 2 - Identity management
- Remove or limit use of local administrator accounts.  Maintain a local account for disaster scenarios, utilize a complex password (preferably rotated automatically) and store it in a secure vault which is only accessible to those who require it.  
- Integrate with an identity store like Microsoft Active Directory (linked to HR processes), ensuring account lifecycle management is in place.

# 3 - Authorization
- Determine which users and systems/tools require access, and what activities they will perform
- Maintain groups for each role and authorize access based on group membership
- Configure all admin interfaces to use these groups for consistent authorization (web ui, device SSH, etc.)
- Apply model of least privileged access

# 4 - Accounting
- Log all authentication/authorization activities
- Log to an external system where data cannot be modified/deleted
- Integrate with a SIEM/Cybersecurity operations team who can detect/respond to unexpected behaviors

# 5 - Conditional access
- Configure conditional access using a SAML solution like Microsoft Azure AD or Okta
- Consider restricting access from managed/corporate computers or devices
- Check the posture of the user/computer being authenticated
- Require multi-factor authentication for any administrative interface (Web, SSH etc.)

# 7 - Vulnerability management process
- Respond quickly to security vulnerabilities
- Setup a notification process with your SD-WAN vendor, integrate into your incident/service request process
- Apply the latest security patches, or software upgrades
- Establish approved software versions, and automated compliance checks

# 6 - System and tools integration
- Consider all types of non-user access, including APIs, SNMP, etc.
- Use modern/secure protocols (e.g. RabbitMQ over syslog, or SNMP version 3 instead of 1/2c)
- Create separate accounts/permissions for each system integration (monitoring, SEIM, reporting, automation etc.)
- Where possible rotate passwords automatically on a regular basis, or establish a manual/periodic password rotate process

# 8 - Penetration test
- Conduct an internal or preferably externally sourced penetration test 
- Liaise with your SD-WAN vendor if they host part of the infrastructure
- Repeat the penetration test periodically, or upon major changes to software or configuration

# 9 - Vendor best practices/hardening
- Ask for assistance from your SD-WAN vendor to apply their best practices for system hardening
- Change all default credentials
- Consider security of related infrastructure such as cloud resources, authentication systems, management/monitoring systems

# 10 - Establish response plans, and conduct drills
- Define response plans for scenarios in the risk mitigation assessment
- Conduct period drills (practical tests, or practice/theoretical situations)
- Ensure the response plan works, and is well known to those involved
