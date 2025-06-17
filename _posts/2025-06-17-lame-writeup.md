---
title:  "Lame - Penetration Testing Write-Up"
layout: post
---

# Lame – Penetration Testing Write-Up

## Introduction

Hello! We've finished up with the Starting Point and are now working our way through some of the active boxes. This particular one was one of the first boxes to come out of HTB. It wasn't very difficult, but it did provide some experience with metasploit's `msfconsole`. On the OSCP we're only allowed to use metasploit on a single machine, so I would prefer to avoid it when possible, but it should still be seen as a tool in our arsenal! This box had a slight red herring with the ftp service, but it didn't go very deep. From what I've researched, you don't want to spend too long on a single service if it doesn't seem very promising as the OSCP is time based and there is a lot to do in those 24 hours. We quickly pivoted to Samba service and found a well know exploit that worked wonders.

---

## Information Gathering

### Target Discovery

Here we started with a more detailed nmap scan `nmap -A {target_ip} -oN scan.initial`

`-A`: Enable OS detection, version detection, script scanning, and traceroute
`-oN`: Output scan in normal

```
Starting Nmap 7.95 ( https://nmap.org ) at 2025-06-13 12:17 EDT
Nmap scan report for 10.10.10.3
Host is up (0.083s latency).
Not shown: 996 filtered tcp ports (no-response)
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.15.2
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 2.6.23 (92%), Arris TG862G/CT cable modem (90%), Belkin N300 WAP (Linux 2.6.30) (90%), Control4 HC-300 home controller or Mobotix M22 camera (90%), Dell Integrated Remote Access Controller (iDRAC6) (90%), Linksys WET54GS5 WAP, Tranzeo TR-CPQ-19f WAP, or Xerox WorkCentre Pro 265 printer (90%), Linux 2.4.21 - 2.4.31 (likely embedded) (90%), Linux 2.4.27 (90%), Linux 2.4.7 (90%), Citrix XenServer 5.5 (Linux 2.6.18) (90%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name: 
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2025-06-13T12:18:47-04:00
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)
|_clock-skew: mean: 2h00m25s, deviation: 2h49m44s, median: 23s

TRACEROUTE (using port 445/tcp)
HOP RTT      ADDRESS
1   82.71 ms 10.10.14.1
2   82.83 ms 10.10.10.3

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 66.67 seconds

```


### Service Enumeration

It does look like we have a few services open
21 - FTP with Anonymous login allowed
22 - SSH
139 & 445 - [Samba smbd](https://en.wikipedia.org/wiki/Samba_(software))
> - `smbd`, which provides the file and printer sharing services, and
> - `nmbd`, which provides the NetBIOS-to-IP-address name service. NetBIOS over TCP/IP requires some method for mapping NetBIOS computer names to the IP addresses of a TCP/IP network.

---

## Enumeration

### Port 21 - FTP

Although we have anonymous login, it doesn't look like we have any file available to download. There might be a thread to pull on here, maybe we will have to upload something malicious or there is a vulnerability in the version. But at first glance, nothing...
![](https://i.imgur.com/216SlEX.png)


I did look to see if we have any exploits, and it does look like we can try a few methods!
![](https://i.imgur.com/vkLcbVR.png)

### Port 139 & 445 - Samba

I try and do a few test connections to see if I was able to extract any information using. I also went ahead and ran this version through `searchsploit` and found a few other exploits we can try as well.
![](https://i.imgur.com/EJ0qaXs.png)

---

## Exploitation

### Port 21 - FTP
I did find a exploit that used metasploit, but at this time I would rather see if there was another way around without using `msfconsole`.

Another exploit that was available seemed promising at first, there was a simple python script. I went ahead and started creating my venv to keep everything clean
`python -m venv .`
`source bin/activate`

I noticed I had to make a few adjustments, and things were just not working super easily. I had to use newer libraries, which given some time I could have figured out, but after a little while, I didn't want to spend more time on it.
![](https://i.imgur.com/xzQhs65.png)

![](https://i.imgur.com/zfJJ3Zh.png)

### Port 139 - Samba
At this point, I didn't see other readily available vulnerabilities other than using `msfconsole` - which is fine, but I would rather not get used to it :) boy was it one heck of a time saver though.

Since we were able to find the CVE before using `searchsploit` we can use:
`msf6 > search cve:2007-2447`
and quickly find an exploit.
![](https://i.imgur.com/pS16eL6.png)

We had to set up some basic parameters using the `set {argument} {value}` command
Followed by `run`.  This gave us a shell, and after `bash -i`, we've got root!
![](https://i.imgur.com/ZPnNO5F.png)


---

## Outcome

Thank you for taking to time to read, or at least skim through this post. This one was pretty easy! It gave some basics on how to use `msfconsole` taught me not stick around too long when there are other avenues to pursue.  

---
