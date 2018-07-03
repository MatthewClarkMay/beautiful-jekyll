---
layout: post
title: CMD Cheat Sheet
tags: [incident-response, pivoting, powershell, windows]
comments: false
---
This page is a command reference for various blue and red team tactics and techniques. I've accumulated most of this content from SANS posters, various blogs, and personal experience. It is meant to be a reference for myself, but hopefully it proves useful for others as well.

[Remote Access](#remote-access)  
[Remote Execution](#remote-execution)  
[Scheduled Task Creation](#scheduled-task-creation)  
[Service Creation](#service-creation)  
[Service Investigation](#service-investigation)  
[WMIEvtConsumers](#wmievtconsumers)  
[WMIC Remote Process Creation](#wmic-remote-process-creation)  
[PowerShell Remoting](#powershell-remoting)  
[Kansa](#kansa)  
[Hunting Persistent Malware](#hunting-persistent-malware)  
[Autoruns](#autoruns)  
[WMIC Process Listing](#wmic-process-listing)
[Netstat Hacks](#netstat-hacks)
[Red Teaming](#red-teaming)
[Configure Windows host as WPA2-PSK Access Point](#configure-windows-host-as-wpa2-psk-access-point)

# Remote Access
### Map admin share using net.exe
```
net use z: \\host\c$ /user:domain\username <password>
```

# Remote Execution

### Scheduled Task Creation
```
at.exe \\host 12:00:00 "c:\temp\maltask.exe"
schtasks.exe /CREATE /TN taskname /TR c:\temp\malware.exe /SC once /RU "SYSTEM" /ST 12:00:00 /S host /U username
```

### Service Creation
```
sc \\host create servicename binpath="c:\temp\malware.exe"
sc \\host start servicename
```

### Service Investigation
```
tasklist /svc
sc config <svc name> type=own
net stop <svc name> && net start <svc name>
```
List services in each svchost block, break a service out of its container, and restart that service in its own container. Great for troubleshooting resource hogging services, discovering services to exploit, or hunting for malware.

If service keeps crashing then check that services recovery options - could be running another program instead of restarting (Kansa module Get-SvcFail.ps1 useful here)

### WMIEvtConsumers
- Event Filter = Trigger condition
- Event Consumer = Script or executable to run
- Binding = Filter + Consumer
- Usually saved to .MOF file
- Triggers can be basically ANYTHING

### WMIC Remote Process Creation
```
wmic /node:host process call create "malware.exe"
Invoke-WmiMethod -Computer host -Class Win32_Process -Name create -Argument "c:\temp\malware.exe"
```

### PowerShell Remoting
Create new PS Session on target, transfer files to or from target, and kill session.
```
$TargetSession = New-PSSession -ComputerName <hostname>
Copy-Item -ToSession $TargetSession -Path <src> -Destination <dst> -Recurse
Copy-Item -FromSession $TargetSession -Path <src> -Destination <dst> -Recurse
Remove-PSSession <id>
```
Enter new session and execute malware on target.
```
Enter-PSSession -ComputerName host
Invoke-Command -ComputerName host -ScriptBlock {Start-Process c:\temp\malware.exe}
```

### Kansa
(placeholder)

# Hunting Persistent Malware

### Autoruns
Filter output based on publisher, eliminate verified known publishers, and examine what remains.
```
C:\autorunsc.exe -accepteula [options] > autoruns.csv

-accepteula    Automatically accept Microsoft software license
-a *           Show all entries
   b           Boot execute
   c           Codecs
   d           Appinit DLLs
   e           Explorer add-ons
   g           Sidebar gadgets (Vista+)
   h           Image hijacks
   i           Internet Explorer add-ons
   k           Known DLLs
   l           Logon startups (default for autorunsc)
   m           WMI entries
   n           Winsock protocol and network providers
   o           Office add-ins
   p           Printer monitor DLLs
   r           LSA security providers
   s           Autostart services and non-disabled drivers
   t           Scheduled tasks
   w           Winlogon entries

-h             Show file hashes
-m             Hide Microsoft signed entries
-s             Verify digital signatures

-t             Show timestamps in normalized UTC
-c             Print output as CSV
-ct            Print output as tab-delimited values
-x             Print output as XML

-vt            Accept VirutTotal terms of service
-v[rs]         Query VirusTotal based on file hash
   r           Opens reports for files with non-zero detection
   s           Uploads files that have never been seen by VirusTotal

-z             Specifies offline Windows system to scan

-nobanner      Do not display startup messages
```

### WMIC Process Listing
```
wmic process list full
```

### Netstat Hacks
```
netstat -nabo 1 | find "<IPADDR or PORT>"
```

# Red Teaming

### Configure Windows host as WPA2-PSK Access Point
```
netsh wlan set hostednetwork mode=allow ssid=<MYSSID> key=<MYPASSWORD> && netsh wlan start hostednetwork
```
