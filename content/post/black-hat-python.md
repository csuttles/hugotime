+++
author = "Chris Suttles"
categories = ["Python", "offsec", "security", "network", "pentesting", "tshark", "wireshark", "HTTP"]
date = 2020-04-17T12:40:59Z
description = ""
draft = false
cover = "/images/2020/04/bhp.jpg"
slug = "black-hat-python"
tags = ["Python", "offsec", "security", "network", "pentesting", "tshark", "wireshark", "HTTP"]
title = "Black Hat Python"

+++


I've been reading and coding my way through the book ["Black Hat Python" by Justin Seitz](https://nostarch.com/blackhatpython) and really enjoying it. I'm only about halfway through it s far, but I've enjoyed it so much I wanted to share my experience so far.

<figure>
       <a href="https://github.com/csuttles/ctlfish">
         <div>
           <div>csuttles/ctlfish</div>
           <div>Tooling and dev doodles related to my activities on hackthebox.eu and other ethical hacking endeavors. - csuttles/ctlfish</div>
           <div>
             <img src="https://github.githubassets.com/favicons/favicon.svg">
             <span>csuttles</span>
             <span>GitHub</span>
           </div>
         </div>
         <div><img src="https://avatars0.githubusercontent.com/u/1102559?s=400&v=4"></div>
       </a>
       <figcaption>You can find the code corresponding to this post in the 'net' directory of this repo</figcaption>
     </figure>

### Checkpoint - Where am I now?

The "Capstone Project" of Chapters 1-4 combines a lot of things in the text that leads up to it. It's not billed as such, but this is how I see it. The book starts with basics of networking and you practice working with sockets through Python and doing a lot of things very manually in the first couple of chapters. This is followed by a whirlwind introduction to [Scapy](https://scapy.net/). Scapy is just terrific and I think this approach was a clever way for the author to demonstrate that. This post is less of a summary of this stuff and will focus just on what I am calling the "Capstone Project".

####

## Capstone

The basic idea is this two step attack which is very 1984 or otherwise potentially nefarious but also a pretty interesting challenge.

1. ARP cache poison attack to insert ourselves as MITM and capture victim traffic to a pcap
2. Parse this capture offline, and extract all the images out of the HTTP traffic present in the data, and then use [opencv](https://opencv.org/) to detect faces within the images, and if found, render an edited image with the detected face highlighted

#### Attack Details

There's a lot going on here, and a lot of details between the start and end of this [attack 'kill chain'](https://medium.com/@winstark_212/cyber-kill-chain-and-mitre-att-ck-c67ab9c59646), so let's look in a little more detail.

ARP Poisoning Attack1. first step of the arp cache poison attack is to get the current state of the later 2 network so we can restore things when the attack is over or cleanup on exit
2. next, we abuse the ARP protocol, and send gratuitous ARPs to the gateway and victim in order to spoof our mac address in each local table. We want the gateway to believe we are the victim, and the victim to believe we are the gateway.
3. now that we are in the middle we must [be configured to forward ip traffic](https://linuxconfig.org/how-to-turn-on-off-ip-forwarding-in-linux)
4. next we [sniff traffic to capture the victim traffic we are passing](https://attack.mitre.org/techniques/T1040/)
5. finally, when we are happy with the traffic collected, we stop sniffing, and restore the layer 2 network by sending simple 'who-has' broadcasts to make the real hosts respond with their correct MAC addresses. There's also an arg to stop capturing after `--count` packets.

Pic Carver1. first step of this is parsing the pcap with scapy, which is easy enough
2. next, we dissect the pcap into tcp sessions so we can follow tcp streams
3. then we walk the packets in the session and grab just the http traffic
4. now we parse out the headers and use that to infer where there is image data
5. since we found image data, we reassemble it from the tcp stream and http response (or request) using scapy
6. next we write the image data to disk as a standard image
7. now we start the second phase of the program and check the image we just wrote to see if it contains a face via opencv
8. if we detected a face in the picture, we use opencv again, this time to draw a rectangle on the image and indicate the face we detected
9. finally we write the image to disk as an edited 'face' image
10. once the whole pcap and all pictures are processed, we print out a summary and exit



# Pics or it didn't happen

The say a picture is worth a thousand words so here we go.

```bash
writing facial recognition edited image to: ./faces/bigface.pcap-pic_carver_face_50.jpeg
writing original image to: ./pictures/bigface.pcap-pic_carver_50.jpeg
```

{{< figure src="/content/images/2020/04/bigface.pcap-pic_carver_50.jpeg" caption="image extracted from pcap" >}}

{{< figure src="/content/images/2020/04/bigface.pcap-bigface.pcap-pic_carver_50.jpeg" caption="facial recognition edit&nbsp;" >}}

#### Isn't this book old?

Yeah. Copyright is 2015, and here we are 5 years later. Python has changed a bit, and so has scapy. That happens.

I think I got a little extra from the book and exercises _because_ it is a little older. For simpler things, all the examples worked with little or no modification in Python 3 (they were originally written for 2). This made it easy to jam in the code from the examples into an IDE and see things work, and start building a foundation without much friction. When I got to the more ambitious examples that strayed from stdlib stuff, things started to break. This meant that I ended up spending more time reading the docs and debugging things, and I think that experience really helps cement a concept.I ended up rewriting the core parsing of the pcap with scapy for the capstone project and that was a great, practical introduction to scapy. I was able to delete a function or two and reduce the code complexity a bit, thanks to [the addition of the http layer into scapy](https://scapy.readthedocs.io/en/latest/layers/http.html); this was not available when the book was published. Now that it is a part of scapy though, it didn't make sense to manually unwind packets and use byte ranges and slices to carve out headers when [you can instead use an instance of the HTTP layer](https://scapy.readthedocs.io/en/latest/api/scapy.layers.http.html) and ask for parts of the packet via attributes. I also re-wrote some of the simpler examples using [asyncio](https://docs.python.org/3/library/asyncio.html), and replaced some of the synchronous socket based stuff using the [higher level streams approach to networking with asyncio](https://docs.python.org/3/library/asyncio-stream.html). All of this made the experience of reading this book even more rewarding for me, because I ended up learning a lot that was not in the book!

#### Want to try it yourself?

Take a peek at the [README](https://github.com/csuttles/ctlfish/tree/master/net#arperpy) in for this stuff. Grab the code and start playing around with your network! Take the general structure of this and change it to do different things; you can easily incorporate some different processing of the photos or any other data you might want to extract. One tip I recommend it to try simple things first. I made sure to find a website that was http only to make the parsing a little easier. Surprisingly, [http://www.ucla.edu/](http://www.ucla.edu/) is all http only so it was easy to just click around their site and gather some traffic I knew would include pictures.

# Thank you

I'd like to thank the author of the book for such a great read and interesting project. I'm really enjoying it and looking forward to the remaining chapters. I don't think I would have tried something like this if it weren't for this book.

I'd also like to thank you, the reader. If you have stuck with me all this way, congratulations! :) Please reach out to me on Twitter if you have any suggestions for stuff you'd like to see or feedback for me! :D

