---
title: "SecDojo - P3t3rP4n"
author: [S4sc0]
subtitle: "N4gy machine write up"
date: "2023-01-18"
keywords: [Network, Security, CTF, Hacking]
lang: "en"
titlepage: true
titlepage-text-color: "FFFFFF"
titlepage-color: "243763"
titlepage-rule-color: "8ac53e"
...


# Information

- **Name:** P3t3rP4n Lab - N4gy Machine
- **Profile:** SecDojo
- **Difficulty:** Easy
- **Description:** The purpose of this lab is to show how an out-of-date network monitoring software can be exploited by an attacker to first obtain unprivileged access as 'apache' user and then exploit another vulnerability in the same network appliance to achieve root access


# Enumeration

## Nmap

Began an Nmap scan

```console
$ nmap -sC -sV 172.16.1.54
Starting Nmap 7.92 ( https://nmap.org ) at 2023-01-15 17:00 UTC
Nmap scan report for 172.16.1.54
Host is up (0.00016s latency).
Not shown: 995 closed tcp ports (conn-refused)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 fc:89:ab:e1:5f:98:4a:12:0b:b2:3e:5c:27:44:4d:71 (RSA)
|   256 f3:15:ce:31:2b:35:a7:18:e4:44:19:38:d8:67:ba:d5 (ECDSA)
|_  256 f5:f7:86:9a:a7:02:94:01:39:0c:52:4a:c7:b0:93:36 (ED25519)
25/tcp  open  smtp     Postfix smtpd
|_smtp-commands: ip-172-31-43-168.us-east-2.compute.internal, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=ip-172-31-43-168.us-east-2.compute.internal
| Subject Alternative Name: DNS:ip-172-31-43-168.us-east-2.compute.internal
| Not valid before: 2019-10-14T13:03:41
|_Not valid after:  2029-10-11T13:03:41
80/tcp  open  http     Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Nagios XI
|_http-server-header: Apache/2.4.29 (Ubuntu)
389/tcp open  ldap     OpenLDAP 2.2.X - 2.3.X
443/tcp open  ssl/http Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Nagios XI
|_http-server-header: Apache/2.4.29 (Ubuntu)
| ssl-cert: Subject: commonName=172.31.43.168/organizationName=Nagios Enterprises/stateOrProvinceName=Minnesota/countryName=US
| Not valid before: 2019-10-14T13:43:57
|_Not valid after:  2029-10-11T13:43:57
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
Service Info: Host:  ip-172-31-43-168.us-east-2.compute.internal; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.62 seconds
zsh: segmentation fault  nmap -sC -sV 172.16.1.54
```

There's couple of interesting ports one of them is port **80** which runs **Apache 2.4.29** web server.

![](./Figure%2001.png)
![](./Figure%2002.png)

**Figure 1:** *This page was displayed after I've clicked on Access Nagois XI button*


# Exploitation

After looking for vulnerabilities of one of the services mentioned by nmap, I've found that Nagios XI which is a network monitoring software is vulnerable to two vulnerablities **CVE-2018-15708** which allows for unauthenticated remote code execution and **CVE-2018-15710** which allows for local privilege escalation.

> "A critical vulnerability exists in the MagpieRSS library. This library contains a custom version of the Snoopy component which allows a remote, unauthenticated attacker to inject arbitrary arguments into a "curl" command. By requesting magpie_debug.php with a crafted value specified in the HTTP GET 'url' parameter, the vulnerable component can be exploited to write arbitrary data to a location on disk that is writable by the 'apache' user. For instance, the location /usr/local/nagvis/share/ is writable and publicly accessible. If an attacker were to write PHP code to this location, arbitrary code execution may be achieved with the privileges of the apache user.
Combined with the local privilege escalation vulnerability, arbitrary code execution with root privileges is feasible."

**Source :** https://www.tenable.com/security/research/tra-2018-37

We can easly exploit those two vulnerabilities with a module called `exploit/linux/http/nagios_xi_magpie_debug` in *metasploit*.

![](./Figure%2003.png)

**Figure 2:** *after setting the necessary options that's our module running*

![](./Figure%2004.png)

**Figure 3:** *We got in as *root*!! and that's our meterpreter*

## Root Flag

![](./Figure%2005.png)

**Figure 4:** */root/proof.txt*

**Flag:** **`Nagy_Sesco-6z6l7dy7dgulhq696ldpd8q0bliyjry3`**