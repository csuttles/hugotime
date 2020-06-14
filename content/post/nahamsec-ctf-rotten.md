---
title: "Nahamcon CTF - Rotten"
date: 2020-06-13T20:15:45-07:00
draft: false
author: "Chris Suttles"
description: ""
slug : "nahamcon-ctf-rotten"
title:  "Nahamcon CTF"
aliases:  ["/nahamcon-ctf-rotten/"]
tags: ["CTF", "Python"]
categories: ["CTF", "Python"]
cover: "/images/2020/06/naham_banner.png"


---

I see a lot of high quality content from the people that put on Nahamcon so I was excited to participate in the CTF. I didn't get as much time to spend on it as I hoped, but I did get a chance to do the scripting challenge "Rotten".

I found a good write up for this challenge which is basically the same approach but with pwntools: https://github.com/csivitu/CTF-Write-ups/tree/master/NahamCon%20CTF/Scripting/Rotten

The code for this walkthrough is here: https://github.com/csuttles/nahamcon

## Where to start

I connected to the host and port defined in the challenge via netcat, and see:

{{< highlight bash >}}
ctlfish@jabroni ~ $ nc jh2i.com 50034
send back this line exactly. no flag here, just filler.
{{< / highlight >}}

I tried this a couple times and start to see what looks like rot13. I fired up burp and dump the string in decoder, but no luck. Since burp only supports rot13 by default, I thought it would be a good idea to try shifting by other increments as well. In a short matter of time, I have an iPython going and I get a socket and start pulling the replies and brute forcing the cipher to get the next step. I got a few characters of the flag after successfully brute forcing and seeing output like this:

{{< highlight python >}}
'10 - 'send back this line exactly. character 23 of the flag is 'c'
{{< / highlight >}}

## How do we reach our goal

Now I have a picture of how this works. We connect and read data. That data might be encoded via a [Caesar Cipher](https://en.wikipedia.org/wiki/Caesar_cipher). If it is, we need to decode it and read a random character and position of the flag. We also might receive a plaintext string. Once we have plaintext (whether we decoded to get it or not) we need to send that back to get out next chunk of data. Once we repeat this process long enough, we eventually decode every character and position pair of the flag, and we submit it to claim our points.

## Automating it

### Socket to me

The first bit is to get a connection. I got everything wired up with the socket module, and stuffed some of that into a little helper function:

{{< highlight python >}}
def newclient(host, port):
    client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    client.connect((socket.gethostbyname(host),port))
    return client

def recvall(sock):
    BUFF_SIZE = 4096 # 4 KiB
    data = b''
    while True:
        part = sock.recv(BUFF_SIZE)
        data += part
        if len(part) < BUFF_SIZE:
            # either 0 or end of data
            break
    return data
{{< / highlight >}}

Early on, I thought I might need to bounce around to different ports or something so I wanted to wrap the creation of the client. I also added a little helper function to make sure I read all the data intended (recvall). This is because if you try to `client.recv()` more data that the buffer of your TCP stack can hold, you will get only a chunk the size of your buffer, so we loop around to make sure we get all the data, but don't accidentally `client.recv()` too many times, because if we get it wrong, the default behavior is to wait for the read and we can easily end up accidentally deadlocked with the other side of the TCP connection. Also writing that loop all the time is clunky and not DRY so there ya go.

### Good devs borrow, great devs steal

I knew that I needed to break Caesar cipher, and I also don't believe that is a wheel _I_ need to reinvent, so I straight up ripped this off from a `site:github` google dork:

{{< highlight python >}}
def decrypt(n, ciphertext):
    """Decrypt the string and return the plaintext"""
    result = ''

    for l in ciphertext:
        try:
            if l.isupper():
                index = LETTERS.index(l)
                i =  (index - n) % 26
                result += LETTERS[i]
            else:
                index = letters.index(l)
                i =  (index - n) % 26
                result += letters[i]

        except ValueError:
            result += l

    return result
{{< / highlight >}}

I edited a couple things to make it Python3 compliant and also avoid using some reserved words. I'm somehow unable to find my source now. Since I am not sure where I got this, I'll just say "thanks internet!"

### Brains

The meat of what makes this all go is the combination of flow control in main and the vars set via return values from parseresp. This is what makes all the important decisions about what to send and builds the flag. The whole thing is something like this pseudo code:
 
{{< highlight python >}}
# main
client = connect
while true:
    resp = client.recv()
    for l in a-z:
        dec = decode(l, resp)
        req = parse_response(resp)
        if req = good:
            flag = update_flag(dec)
            client.send(req)
{{< / highlight >}}

## Baking a cake

So here is how we put all these ingredients together and bake the cake. I use a couple magic methods in iPython for development during CTFs:

* The first thing I do is `%load_ext autoreload`
* Next, `%autoreload 1` - this sets the extension to automatically reload any module loaded with `%aimport`
* Now, import our module for the CTF so it is reloaded as we edit via `%aiport rotten`

Now when I edit the file I keep in git, iPython sees the file is updated, and automatically reloads my changes, so I can test with a simple `rotten.main()`

Here's what that looks like, and we can see the flag magically get assembled before our eyes:

<script id="asciicast-HGlcctJmWOEBBFDtM4ALiUi4N" src="https://asciinema.org/a/HGlcctJmWOEBBFDtM4ALiUi4N.js" async></script>

## Thanks!

Thank you for a fun challenge and for all you do for the infosec community! You don't know me, but I've learned a lot from you!

[@_JohnHammond](https://twitter.com/_johnhammond)
[@NahamSec](https://twitter.com/NahamSec)
[@STOKFredrik](https://twitter.com/stokfredrik)
[@TheCyberMentor](https://twitter.com/thecybermentor)

Thanks If you read this far, I hope you enjoyed it, and I hope it is useful to you. :)