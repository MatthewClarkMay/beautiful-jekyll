---
layout: post
title: Security Monitoring Checklist
tags: [intrusion-detection] 
comments: true
---

I recently listened to Eric Conrad speak at SANS in Orlando, and many of the things he discussed have made me take a step back to re-evaluate the way we should be monitoring our organizations. Below is a checklist you can use to begin effectively monitoring your network with entirely free tools. Some of this was originally mentioned [here](http://www.ericconrad.com/2016/10/quality-not-quantity-talk-commands-and.html) by Eric Conrad, but I have summarized particular snippets of his talk, and added my own input as well.

# Intelligently Store/Analyze Data:
- Centralized Log Storage (SIEM):
  - [ELK Stack](https://www.elastic.co/):
    - No limit on number of log sources
    - The stack is free and open source, but the [X-Pack](https://www.elastic.co/guide/en/x-pack/current/xpack-introduction.html) package is not. This package adds security, monitoring, and alerting features not offered with the open source components. You can download it and try a 30 day trial before purchasing.
    - Elastic offers a license by node plan, rather than licencing by events per second like many vendors.
    - Fast querying of only the most important logged events.
  - Also take a look at [SOF-ELK](https://github.com/philhagen/sof-elk), an attempt headed by Phil Hagen to distribute an ELK VM Appliance for security monitoring.
- Windoes Event Collector: Can be used to easily collect logs (Sysmon, etc.) which can then be shipped to your ELK stack using the [Beats Data Shippers](https://www.elastic.co/products/beats). I will be posting a tutorial for this configuration in time.
  - Couple of decent resources below:
    - https://msdn.microsoft.com/en-us/library/bb427443(v=vs.85).aspx
    - https://www.petri.com/configure-event-log-forwarding-windows-server-2012-r2
    - https://www.loggly.com/ultimate-guide/centralizing-windows-logs/
- DeepBlueCLI - https://github.com/sans-blue-team/DeepBlueCLI

# Event Logging:
- High-entropy service names are highly suspicious:
  - Service Name: MmvTBipnvXFMNFUs
  - Service File Name %SYSTEMROOT%\llTTAajm.exe

## Events to Watch:
- System Event ID 7045 - Service Creation
- System Event ID 7030 - Service Creation Errors
- Security Event 4720 "A user account was created"
- Security Event 4732 ("A member was added to a security-enabled local group")
- Security Event 4728 ("A member was added to a security-enabled global group")
- Full Command Line Logging of all Processes:
  - gpedit.msc: set the following variables:
    - Computer Configuration\Windows Settings\Security Settings\Advanced Audit Policy Configuration\System Audit Policies\Detailed Tracking
    - Computer Configuration\Administrative Templates\System\Audit Process Creation

## Event Queries:
- Service Creation Query:
  - Get-WinEvent -FilterHashtable @{logname='system';id=7045,7030}
- User Creation/Escalation Query:
  - Get-WinEvent -FilterHashtable @{logname="Security";id=4720,4732,4728}
- Full Command Line Logging Query:
  - Get-WinEvent -FilterGashtable @{Logname="Security"; ID=4688}
- List of abusive windows commands to monitor - http://blog.jpcert.or.jp/.s/2016/01/windows-commands-abused-by-attackers.html

## Sysmon - Enhanced Logging:
- Can be deployed to all endpoints for enhanced logging
  - [Starter Config File](https://github.com/SwiftOnSecurity/sysmon-config)
  - [Config File Tuning](https://medium.com/@lennartkoopmann/explaining-and-adapting-tays-sysmon-configuration-27d9719a89a8)
  - Windows File Path: C:\Windows\System32\winevt\Logs\Microsoft-Windows-Sysmon%4Operational.evtx

## Whitelisting:
- Send these logs to ELK as well
  - AppLocker:
    - 8003: (exe or dll) was allowed to run but would have been prevented from running if AppLocker policy were enforced (audit mode)
    - 8004: (exe or dll) was not allowed to run (enforce mode)
    - Get-WinEvent -FilterHashTable @{LogName="Microsoft-Windows-AppLocker/EXE and DLL"; ID=8003,8004}

## Active Mitigations: 
- EMET BlockLogs:
  - Get-WinEvent -FilterHashtable @{LogName="application"; ProviderName="EMET"; id=2}
- Antivirus - Send these logs to ELK as well
 

## Full Packet Capture + Network Security Monitoring Toolkit - Security Onion:
- [WIKI](https://github.com/Security-Onion-Solutions/security-onion/wiki/IntroductionToSecurityOnion)
