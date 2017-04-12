---
layout: page
title: Blue Team Tools and References
---

Here is a small collection of tools I find useful for various blue team operations.

## Tools
- [DeepBlueCLI](https://github.com/sans-blue-team/DeepBlueCLI) - PowerShell Tool developed by [Eric Conrad](http://www.ericconrad.com/) to parse .evtx files for IOCs. Also check out his [talk](https://drive.google.com/file/d/0ByeHgv6rpa3gWi1xaWhZaWFQSjA/view?usp=sharing) about the deadliest events to be aware of while monitoring, and how you can enable logging for those events in the first place.
- [REMnux](https://remnux.org/) - Free Linux toolkit for assisting malware analysts with reverse-engineering malicious software. It is maintained by [Lenny Zeltser](https://zeltser.com/), and can also be installed on top of your [SIFT Workstation](http://digital-forensics.sans.org/community/downloads)
- [Security Onion](https://securityonion.net/) - Security Onion is a Linux distro for intrusion detection, network security monitoring, and log management. It’s based on Ubuntu and contains Snort, Suricata, Bro, OSSEC, Sguil, Squert, ELSA, Xplico, NetworkMiner, and many other security tools.
- [SIFT Workstation](http://digital-forensics.sans.org/community/downloads) - Free toolkit for incident response and digital forensics.
- [Sysmon](https://technet.microsoft.com/en-us/sysinternals/sysmon) - Windows system service and device driver that, once installed on a system, remains resident across system reboots to monitor and log system activity to the Windows event log. It provides detailed information about process creations, network connections, and changes to file creation time. Also be sure to check out the [Sysmon config file](https://github.com/SwiftOnSecurity/sysmon-config) provided by SwiftOnSecurity. This is a fantastic starting point for security monitoring, and you can tailor it to your own needs.

## References