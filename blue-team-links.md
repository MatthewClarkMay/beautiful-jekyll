---
layout: page
title: Blue Team Tools and References
---

Here is a small collection of tools and references I find useful for building an effective IR infrastructure and toy chest. I'll be adding to this list as time goes on.

# Tools
## Forensics
- [DeepBlueCLI](https://github.com/sans-blue-team/DeepBlueCLI) - PowerShell Tool developed by [Eric Conrad](http://www.ericconrad.com/) to parse .evtx files for IOCs. Also check out his [talk](https://drive.google.com/file/d/0ByeHgv6rpa3gWi1xaWhZaWFQSjA/view?usp=sharing) about the deadliest events to be aware of while monitoring, and how you can enable logging for those events in the first place.
- [REMnux](https://remnux.org/) - Free Linux toolkit for assisting malware analysts with reverse-engineering malicious software. It is maintained by [Lenny Zeltser](https://zeltser.com/), and can also be installed on top of your [SIFT Workstation](http://digital-forensics.sans.org/community/downloads)
- [SIFT Workstation](http://digital-forensics.sans.org/community/downloads) - Free toolkit for incident response and digital forensics.

## Continuous Monitoring & Defense
### Network
- [Security Onion](https://securityonion.net/) - Security Onion is a Linux distro for intrusion detection, network security monitoring, and log management. It’s based on Ubuntu and contains Snort, Suricata, Bro, OSSEC, Sguil, Squert, ELSA, Xplico, NetworkMiner, and many other security tools. The [wiki](https://github.com/Security-Onion-Solutions/security-onion/wiki/IntroductionToSecurityOnion) is a great starting point for those who want to learn more.

### Endpoints
- [AppLocker](https://technet.microsoft.com/en-us/itpro/windows/keep-secure/applocker-overview) - Free and effective application whitelisting for Windows machines. Mentioned in Eric Conrad's [talk](https://drive.google.com/file/d/0ByeHgv6rpa3gWi1xaWhZaWFQSjA/view?usp=sharing) as well
- [EMET (Enhanced Mitication Experience Toolkit)](https://www.microsoft.com/en-us/download/details.aspx?id=50766) is a tool that hardens Windows operating systems against a series of common exploit tactics. Also mentioned in Eric's [talk](https://drive.google.com/file/d/0ByeHgv6rpa3gWi1xaWhZaWFQSjA/view?usp=sharing)
- [Sysmon](https://technet.microsoft.com/en-us/sysinternals/sysmon) - Windows system service and device driver that, once installed on a system, remains resident across system reboots to monitor and log system activity to the Windows event log. It provides detailed information about process creations, network connections, and changes to file creation time. SwiftOnSecurity published a great [Sysmon config file](https://github.com/SwiftOnSecurity/sysmon-config) to use as a starting point. Also refer to [this article](https://medium.com/@lennartkoopmann/explaining-and-adapting-tays-sysmon-configuration-27d9719a89a8) for additional explanation of this config file. 
# References
