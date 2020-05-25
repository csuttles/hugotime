+++
author = "Chris Suttles"
categories = ["AWS"]
date = 2017-10-24T22:52:35Z
description = ""
draft = false
cover = "/images/2017/10/scenario-1-sg-inbound-gwt.png"
slug = "limiting-access-in-security-groups"
tags = ["AWS"]
title = "Limiting Access in Security Groups"
aliases = ["/limiting-access-in-security-groups/"]

+++


Here's a very quick, practical example why it is important to limit access in AWS security groups for your bastion hosts (jump hosts). 

{{< highlight markdown >}}
sudo grep -oP '(?:\d+\.){3}\d+' -h /var/log/secure* | sort |uniq -c | sort -n | awk '{sum +=$1 ; count ++} END {print sum, count}'
65546 1680
{{< / highlight >}}

This is a sum of log lines (auth attempts) per IPv4 address and count of unique IPv4 addresses in /var/log/secure* from an instance with SSH open to the world.

Let's step through this. This is what the matched lines look like, here's the (truncated/sanitized) output of `sudo grep -P '(?:\d+\.){3}\d+'  /var/log/secure*`

{{< highlight markdown >}}
/var/log/secure-20171001:Sep 24 22:27:20 ip-86-75-30-9 sshd[30310]: Address 185.82.216.241 maps to vds-vektort13-89700.itldc-customer.net, but this does not map back to the address - POSSIBLE BREAK-IN ATTEMPT!
/var/log/secure-20171001:Sep 24 22:27:20 ip-86-75-30-9 sshd[30310]: Invalid user admin from 185.82.216.241 port 58118
/var/log/secure-20171001:Sep 24 22:27:20 ip-86-75-30-9 sshd[30310]: Connection closed by 185.82.216.241 port 58118 [preauth]
/var/log/secure-20171001:Sep 24 22:27:21 ip-86-75-30-9 sshd[30312]: Invalid user admin from 162.247.72.200 port 47848
/var/log/secure-20171001:Sep 24 22:27:22 ip-86-75-30-9 sshd[30312]: Connection closed by 162.247.72.200 port 47848 [preauth]
/var/log/secure-20171001:Sep 25 00:33:30 ip-86-75-30-9 sshd[31629]: Invalid user admin from 119.254.153.43 port 54376
/var/log/secure-20171001:Sep 25 00:33:32 ip-86-75-30-9 sshd[31629]: error: maximum authentication attempts exceeded for invalid user admin from 119.254.153.43 port 54376 ssh2 [preauth]
/var/log/secure-20171001:Sep 25 01:48:47 ip-86-75-30-9 sshd[32407]: Received disconnect from 191.96.249.156 port 58456:11: Bye Bye [preauth]
/var/log/secure-20171001:Sep 25 01:48:47 ip-86-75-30-9 sshd[32407]: Disconnected from 191.96.249.156 port 58456 [preauth]
/var/log/secure-20171001:Sep 25 01:48:49 ip-86-75-30-9 sshd[32409]: Invalid user ubnt from 191.96.249.156 port 37724
{{< / highlight >}}

Clearly, script kiddies are scanning and jamming with brute force. If they had done any footprinting, they would not be trying to log in as admin or ubnt. 

We are using the `-P` option and a simple regex to catch lines with IPv4 addresses. It's worth noting this will also grab valid login lines from your own access if you are connecting over IPv4.

So that's what our lines look like, then we add the `-o` option to grep so we can output just the IPv4 address of matching lines. We also add the `-h` option to suppress the filename, so we don't count any source that spans log files more than once. Next, we pipe into the classic UNIX idiom `sort |uniq -c` to sort all lines (just IPv4 addresses at this point), and pull unique entries, with a count for each (`uniq -c`). The `sort -n` is an artifact of me looking as I was building the output interactively, and sorts IPv4 addreses by the number of lines (attempts). I wanted to see where the bulk of these requests originated.

This is what things look like so far. I added a `|tail` so we can see the top offenders from the interactive output:

{{< highlight markdown >}}
sudo grep -oP '(?:\d+\.){3}\d+' -h /var/log/secure* | sort |uniq -c | sort -n | tail
    998 198.98.57.188
   1012 185.165.29.82
   1176 185.165.29.198
   1578 52.172.35.133
   1929 88.247.250.201
   1975 5.189.133.76
   2270 27.118.21.218
   3901 202.112.23.245
   7343 198.98.62.121
  15308 61.255.108.136
 {{< / highlight >}}
 
These are IPv4 adresses from people/bots/attackers trying to breach my instance.

Let's use `whois` to find out more about the top attacker:

{{< highlight markdown >}}
whois 61.255.108.136
[Querying whois.apnic.net]
[Redirected to whois.krnic.net]
[Querying whois.krnic.net]
[whois.krnic.net]
query : 61.255.108.136


# KOREAN(UTF8)

조회하신 IPv4주소는 한국인터넷진흥원으로부터 아래의 관리대행자에게 할당되었으며, 할당 정보는 다음과 같습니다.

[ 네트워크 할당 정보 ]
IPv4주소           : 61.254.160.0 - 61.255.255.255 (/16+/18+/19)
기관명             : 에스케이브로드밴드주식회사
서비스명           : broadNnet
주소               : 서울특별시 중구 퇴계로 24
우편번호           : 04637
할당일자           : 20010724

이름               : IP주소 담당자
전화번호           : +82-2-106-2
전자우편           : ip-adm@skbroadband.com

조회하신 IPv4주소는 위의 관리대행자로부터 아래의 사용자에게 할당되었으며, 할당 정보는 다음과 같습니다.
--------------------------------------------------------------------------------


[ 네트워크 할당 정보 ]
IPv4주소           : 61.255.108.0 - 61.255.108.255 (/24)
기관명             : 에스케이브로드밴드주식회사
네트워크 구분      : CUSTOMER
주소               : 서울 구로구 구로동
우편번호           : 08389
할당내역 등록일    : 20080806

이름               : IP주소 담당자
전화번호           : +82-2-106-2
전자우편           : ip-adm@skbroadband.com

조회하신 IPv4주소는 위의 관리대행자로부터 아래의 사용자에게 할당되었으며, 할당 정보는 다음과 같습니다.
--------------------------------------------------------------------------------


[ 네트워크 할당 정보 ]
IPv4주소           : 61.255.108.0 - 61.255.108.255 (/24)
기관명             : 에스케이브로드밴드주식회사
네트워크 구분      : CUSTOMER
주소               : 서울 구로구 구로동
우편번호           : 08389
할당내역 등록일    : 20080806

이름               : IP주소 담당자
전화번호           : +82-2-106-2
전자우편           : bbb@aaa.com


# ENGLISH

KRNIC is not an ISP but a National Internet Registry similar to APNIC.

[ Network Information ]
IPv4 Address       : 61.254.160.0 - 61.255.255.255 (/16+/18+/19)
Organization Name  : SK Broadband Co Ltd
Service Name       : broadNnet
Address            : Seoul Jung-gu Toegye-ro 24
Zip Code           : 04637
Registration Date  : 20010724

Name               : IP Manager
Phone              : +82-2-106-2
E-Mail             : ip-adm@skbroadband.com

--------------------------------------------------------------------------------

More specific assignment information is as follows.

[ Network Information ]
IPv4 Address       : 61.255.108.0 - 61.255.108.255 (/24)
Organization Name  : SKBroadband Co. Inc
Network Type       : CUSTOMER
Address            : Seoul Guro-gu Digital-ro 30-gil
Zip Code           : 08389
Registration Date  : 20080806

Name               : IP Manager
Phone              : +82-2-106-2
E-Mail             : bbb@aaa.com



- KISA/KRNIC WHOIS Service -
{{< / highlight >}}

I'm probably never going to log in from Korea. If I ever find myself in Korea, it's likely I would be using a VPN anyway, so I feel pretty comfortable with blocking the range of this attacker. We'll get to that part later.

Lastly, we pipe to another UNIX classic `awk '{sum +=$1 ; count ++} END {print sum, count}'`. Here, awk is generating a sum of attempts based on the first field (`sum`), and incrementing a counter (`count`) once for each line while reading the input. The `END` block prints the values of the `sum` and `count` variables, giving us the final count of attempts over IPv4 and unique IPv4 addresses.

The fix for this is very simple; Restrict access to instances from limited IP ranges using Security Groups. By looking at valid logins in the output of the `last` command and a trivial amount of additional parsing of `/var/log/secure*`, it is very easy to find the ranges I actually use to legitimately access this instance. I actually already knew these ranges, since they are present in other security groups, but that's a good way to find them if you need to tune up your security by paring down access. Once you specify the legitimate ranges for your SSH access in your Security Groups, other ranges are blocked implicitly, including all of the attackers we found through parsing `/var/log/secure*` in this example.

After cutting down the range for ssh access to this same instance, some time and a log rotation, the output of the command at the beginning of this post is now empty.

### AWS Security Best Practices

You can download the whitepaper [here](https://aws.amazon.com/whitepapers/aws-security-best-practices/)

Note that this entire post is just an example of a single bullet point in this paper:

> *  Restrict access to instances from limited IP ranges using Security Groups

