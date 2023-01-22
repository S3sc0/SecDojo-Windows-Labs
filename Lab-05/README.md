---
title: "SecDojo - Kn4ck 17"
author: [S4sc0]
subtitle: "Un4ckn0wl3dg3d machine write up"
date: "2023-01-18"
keywords: [Network, Security, CTF, Hacking]
lang: "en"
titlepage: true
titlepage-text-color: "FFFFFF"
titlepage-color: "243763"
titlepage-rule-color: "8ac53e"
...


# Information

- **Name:** Kn4ck 17 Lab - Un4ckn0wl3dg3d Machine
- **Profile:** SecDojo
- **Difficulty:** Medium
- **Description:** The purpose of this lab is to learn how to perform network packet analysis in order to get more intel about the hidden network infrastructure, detect misconfigurations, and most importantly, how you can make use of this intel to get access to systems and escalate your privileges.

# Enumeration

## TCP Nmap Scan

Began with an nmap scan.

```console
# Nmap 7.92 scan initiated Thu Jan 19 11:28:38 2023 as: nmap -sC -sV -T4 -oA 195nmap 172.16.1.195
Nmap scan report for 172.16.1.195
Host is up (0.00017s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT   STATE    SERVICE VERSION
21/tcp filtered ftp
22/tcp open     ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 6c:0e:3e:0a:a3:1d:43:7b:f0:27:53:36:d3:82:e3:b7 (RSA)
|   256 9f:66:9b:d7:5b:ca:c4:df:31:42:21:73:df:e1:19:bd (ECDSA)
|_  256 3b:e5:47:86:27:0e:b3:7f:65:83:82:e3:75:ea:37:f4 (ED25519)
80/tcp open     http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

As you can see we have ports **22** and **80** open and **21** protected with a firewall apparently.

## UDP Nmap Scan

Running UDP scan with nmap to see if there's any available service that uses UDP Protocol.

```console
# Nmap 7.92 scan initiated Thu Jan 19 17:40:10 2023 as: nmap -sU -T4 -sC -sV -oN udpscan 172.16.1.195
Nmap scan report for 172.16.1.195
Host is up (0.000095s latency).

PORT    STATE SERVICE VERSION
161/udp open  snmp    SNMPv1 server; net-snmp SNMPv3 server (public)
| snmp-info:
|   enterprise: net-snmp
|   engineIDFormat: unknown
|   engineIDData: 111ca1399bf4665d00000000
|   snmpEngineBoots: 19
|_  snmpEngineTime: 35m43s
| snmp-sysdescr: Linux ip-172-16-1-195 5.3.0-1019-aws #21~18.04.1-Ubuntu SMP Mon May 11 12:33:03 UTC 2020 x86_64
|_  System uptime: 35m42.68s (214268 timeticks)
MAC Address: 0E:50:D3:81:4D:F2 (Unknown)
Service Info: Host: ip-172-16-1-195
```

I've found **SNMP** service running on port **161** which is interesting! SNMP stands for Simple Network Monitoring Protocol which helps in monitoring devices in a network.

> "SNMP runs on the application layer and consists of a SNMP manager and a SNMP agent. The SNMP manager is the software that is running on a pc or server that will monitor the network devices, the SNMP agent runs on the network device."

> "The database that I just described is called the MIB (Management Information Base) and an object could be the interface status on the router (up or down) or perhaps the CPU load at a certain moment. An object in the MIB is called an OID (Object Identifier)."

**Source:** https://networklessons.com/cisco/ccie-routing-switching/introduction-to-snmp

We can get those OIDs in plain text since public community string is enabled, we'll use `snmpwalk` utility to do that.

```console
$ snmpwalk -v1 -c public 172.16.1.195
iso.3.6.1.2.1.1.1.0 = STRING: "Linux ip-172-16-1-195 5.3.0-1019-aws #21~18.04.1-Ubuntu SMP Mon May 11 12:33:03 UTC 2020 x86_64"
iso.3.6.1.2.1.1.2.0 = OID: iso.3.6.1.4.1.8072.3.2.10
iso.3.6.1.2.1.1.3.0 = Timeticks: (292262) 0:48:42.62
iso.3.6.1.2.1.1.4.0 = STRING: "FTP service started"
iso.3.6.1.2.1.1.5.0 = STRING: "ip-172-16-1-195"
iso.3.6.1.2.1.1.6.0 = STRING: "Sitting on the Bay 7000, 8000, 9000 ...."
...
```

