---
layout: post
title: Ten SD-WAN Identity and Access Management Security Considerations
date: 2021-10-23 07:38:20 +0000
description: 
img: sd-wan.jpg
tags: [sd-wan, access, control, identity, management, tacacs, saml]
---

I’ve been working on commercialization of a new SD-WAN solution, including identity and access management for administrative users and system/tools integration. In this article, I’ll document some of the key considerations for securing access to any SD-WAN solution.

Ultimately, the goal is to ensure the security and integrity of the network infrastructure, by identifying risks and establishing controls to reduce the likelihood and severity of CyberSecurity incidents.

# 1 - Risk assessment
- Liaise with your Cybersecurity team from the outset
- Completing a risk assessment before deploying SD-WAN ensures appropriate controls are implemented from the beginning
- The risk assessment should identify potential losses/failures, and determine control measures that will reduce/prevent a condition from occurring, or limit its severity
- Work with your selected vendor to get their input/suggestions
- Review the completed assessment with your CyberSecurity team and other stakeholders

# 2 - Identity management
- Integrate with a well-maintained external identity store (could be Microsoft Active Directory using SAML/OAuth2 or RADIUS)
- Ensure account lifecycle management is in place, ideally linked to HR processes
- Avoid use of local user accounts, or limit use to situations where external identity stores are unreachable
- If using local accounts, use a password with high entropy (preferably rotated automatically) and store credentials in a secure vault which is only accessible to those who require it
- Prefer 'remote then local' authentication order (local accounts should only be used when external identity stores are unreachable)

# 3 - Authorization
- Determine which users and systems/tools require access, and what specific activities they need to perform
- Maintain separate groups for each user role and authorize access based on group membership
- Configure all administrative user interfaces to use these groups for consistent authorization (Web, SSH, APIs, etc.)
- Apply least privileged access model when determining permissions for each role

# 4 - Accounting
- Log all authentication/authorization activities, and consider an appropriate data retention policy
- Store logs in an external system where data cannot be modified/deleted
- Integrate with a SEIM product and your enable your Cybersecurity operations team to detect/respond to unexpected behaviors

# 5 - Conditional access
- Configure conditional access using a solution like Microsoft Azure AD or Okta
- Consider only allowing access from managed/corporate computers or devices
- Consider checking the posture of the user/computer being authenticated
- Require multi-factor authentication for all interactive administrative interfaces (Web, SSH, etc.), especially for public cloud-hosted SD-WAN solutions.

# 6 - Vulnerability management process
- Respond quickly to security vulnerabilities by defining and establishing a vulnerability management process
- Setup a notification system with your SD-WAN vendor, integrate it into your incident/service request processes
- Apply the latest security patches, or software upgrades in a timely manner
- Establish approved software versions, and automated compliance checks

# 7 - System and tools integration
- Consider all types of non-user/interactive access, including APIs, SNMP, etc.
- Use modern/secure protocols (e.g. RabbitMQ over syslog, or SNMP version 3 instead of 1/2c)
- Implement controls to limit where service accounts can connect from/to
- Create separate accounts/permissions for each system integration (monitoring, SEIM, reporting, automation, etc.)
- Where possible rotate passwords automatically on a regular basis or establish a manual/periodic password rotation process

# 8 - Penetration test
- Conduct an internal or preferably external (independent) penetration test
- Liaise with your SD-WAN vendor/hosting provider before conducting tests
- Repeat penetration tests periodically, or upon major changes to software or configuration

# 9 - Vendor best practices/hardening
- Find out security best practices from your SD-WAN vendor
- Change all default credentials/apply recommended hardening
- Consider the security of related infrastructure such as cloud resources, authentication/management/monitoring systems

# 10 - Establish response plans, and conduct drills
- Define response plans for scenarios identified in the risk assessment
- Conduct periodic drills (practical tests, or practice/theoretical situations)
- Ensure the response plan works and is well known/understood to those involved
