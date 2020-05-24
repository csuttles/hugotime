+++
author = "Chris Suttles"
categories = ["IPv6", "IPv4", "network"]
date = 2018-01-28T14:15:37Z
description = ""
draft = false
cover = "/images/2018/01/Screen-Shot-2018-01-27-at-3.53.00-PM.png"
slug = "a-brief-comparison-of-ipv4-and-ipv6"
tags = ["IPv6", "IPv4", "network"]
title = "A Brief Comparison of IPv4 and IPv6"

+++


# IPv4 and IPv6

This post will explore key differences between IPv4 and IPv6. IPv4 has been around for a long time and is what moves data around the internet. IPv6 has also been around for many years at this point, but is finally seeing adoption now that IPv4 addresses are nearly exhausted world wide. There are many papers and posts discussing why adoption of IPv6 is important, so I will instead focus on the technical details and leave the reader to make their own evaluation of the merits of IPv6 adoption. This post is a little different from my usual posts; it is my final project in a course at UC Berkeley Extension: [COMPSCI X433 Fundamentals of Data Communications and Networking](https://extension.berkeley.edu/search/publicCourseSearchDetails.do?method=load&courseId=40657#courseSectionDetails_43469550).

## Address Space


IPv6 has 340,282,366,920,938,463,463,374,607,431,768,211,456 (2^128) total addresses.
IPv4 as 4,294,967,296 (2^32) total addresses.

[RFC 4291](https://tools.ietf.org/html/rfc4291) is the latest specification for IPv6 addressing at the time of this writing. Section 2 discusses the IPv6 address format in great detail (RFCs are a great source of this kind of detail).

While the RFC is an excellent reference, the key difference in the number of addresses is the length of addresses in IPv6 vs IPv4. IPv6 uses 128 bits for addresses, while IPv4 uses only 32 bits.

## No NAT is Better

Large and small networks have needed to resort to complex routing and NAT (Network Address Translation) in order to stretch IPv4 resources, and IPv6 solves some interesting challenges faced with IPv4 (like NAT). [RFC 3022](https://tools.ietf.org/html/rfc3022) is a very good overview of the complexity involved in even a simple "Traditional NAT", where the primary concern is unidirectional traffic from the private (NAT'd) network, out to the public network (the Internet). [RFC 2663](https://tools.ietf.org/html/rfc2663) is a good overview of other types of NAT. Even the casual home user with a single IPv4 address obtained from their ISP is reliant on NAT of some kind, though they may not be aware; my mother, for example, doesn't know she uses and needs NAT.

This is an easily overlooked area where IPv6 shines. The IPv6 address architecture specification in [RFC 4291, section 2.5.4](https://tools.ietf.org/html/rfc4291#section-2.5.4) defines Global Unicast Adrresses (GUA). These addresses are globally routable, meaning that every device anywhere with a GUA is routable from any other device with a GUA. GUAs can still be private, in the sense that you can restrict what traffic is allowed to reach them via firewall. This means that every router, including those serving a home network, will not need to maintain the complex mappings of ports and addresses that are necessary for even the most simple NAT (Traditional NAT). There's no need for those devices to burn processor cycles (and therefore power) to examine and selectively rewrite headers in every packet that traverses the device. The routing just works.

## More Efficient Headers

Here's and example of an [IPv4 Header:](https://tools.ietf.org/html/rfc791#section-3.1)
{{< highlight markdown>}}
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |Version|  IHL  |Type of Service|          Total Length         |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |         Identification        |Flags|      Fragment Offset    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  Time to Live |    Protocol   |         Header Checksum       |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                       Source Address                          |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Destination Address                        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Options                    |    Padding    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
{{< / highlight >}}

Here's an example of an [IPv6 Header Format:](https://tools.ietf.org/html/rfc8200#section-3)
{{< highlight markdown>}}
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |Version| Traffic Class |           Flow Label                  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |         Payload Length        |  Next Header  |   Hop Limit   |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                                                               |
   +                                                               +
   |                                                               |
   +                         Source Address                        +
   |                                                               |
   +                                                               +
   |                                                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                                                               |
   +                                                               +
   |                                                               |
   +                      Destination Address                      +
   |                                                               |
   +                                                               +
   |                                                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
{{< / highlight >}}
It's important to note that both of these ASCII representations are 32 bits wide. It's very easy to see how much simpler the IPv6 header is in structure. While the large addresses do make it larger than an IPv4 header, it is much less complicated, and it was designed with efficiency in mind.

This [excellent pdf from ripe](https://www.ripe.net/participate/meetings/roundtable/february-2009/RobBlokzijlroundtable2009Rob2.pdf) covers some of the key improvements in the IPv6 header, which include:

* No options, which also means the header is a fixed length
* No header checksum, which means no checksum calculation at each hop
* No fragmentation field (options, fragmentation both move to extension header)

The elimination of the header checksum in IPv6 is a very important change, since the header checksum in IPv4 is redundant; there is already error checking and correction the [data link layer](https://www.cisco.com/cpress/cc/td/cpress/fund/ith/ith01gb.htm#xtocid1668412), so checking the IPv4 header for every packet when the underlying Ethernet frame is already handling error checking is unnecessary and very inefficient. My home router is capable of handling 2 million packets per second. If you extrapolate the number of routers and switches that are aggregated to form the Internet, and consider the fact that these devices are almost invariably  more expensive and powerful than the router I use at home, it is easy to see how this small change has a big impact. If every device in the path of a packet no longer needs to process unnecessary checksums for *at least* trillions (probably exponentially more) packets, that's a tremendous efficiency gain.


## Link Local Networks and SLAAC

Link local addresses in IPV6 are for use on a single link. Here's the definition from [RFC 4291, section 2.5.6](https://tools.ietf.org/html/rfc4291#section-2.5.6):
{{< highlight markdown>}}
   Link-Local addresses are for use on a single link.  Link-Local
   addresses have the following format:

   |   10     |
   |  bits    |         54 bits         |          64 bits           |
   +----------+-------------------------+----------------------------+
   |1111111010|           0             |       interface ID         |
   +----------+-------------------------+----------------------------+

   Link-Local addresses are designed to be used for addressing on a
   single link for purposes such as automatic address configuration,
   neighbor discovery, or when no routers are present.

   Routers must not forward any packets with Link-Local source or
   destination addresses to other links.
{{< / highlight >}}

This facet of IPv6 is much more effective in my experience than the IPv4 counterpart, as defined in [RFC 3927](https://tools.ietf.org/html/rfc3927). In IPv4, the typical model of single addresses per host means that hosts typically get a link local address through APIPA, where DHCP fails, the host gives up, and assigns an address in the 169.245.0.0/16 range. With IPv6, however, the addressing model is designed to accommodate multiple addresses at every endpoint, so even hosts that are configured with a GUA (Global Unicast Address) also have a link local address. This means that getting packets around via only link local addressing is much more likely to succeed, since link local addressing in IPV6 includes *all hosts*, where IPv4 link local addressing likely includes only hosts that are misconfigured.

## Stateless Automatic Address Configuration (SLAAC)

While IPv4 relies on manual address configuration on DHCP for host addressing, IPv6 offers stateless automatic address configuration (SLAAC), defined in [RFC 4862](https://tools.ietf.org/html/rfc4862). This allows hosts to dynamically configure their own addressing. Through Router Advertisements  and solicitation (as described in [RFC 4862 section 5](https://tools.ietf.org/html/rfc4862#section-5.5.1)), hosts are able to detect the network prefix and gateway, configuring themselves for access both within the local network and to routed networks (like the Internet) without any manual configuration or a DHCP server. The SLAAC specification includes important features like [duplicate address detection](https://tools.ietf.org/html/rfc4862#section-5.4), which is accomplished through the use of [neighbor solicitation messages](https://tools.ietf.org/html/rfc4862#section-5.4.2), which is part of [NDP (neighbor discovery protocol)](https://tools.ietf.org/html/rfc4861). NDP is essentially the IPv6 replacement for IPv4's [ARP (address resolution protocol)](https://tools.ietf.org/html/rfc826).

## Prefix Delegation

With IPv6 it's possible to obtain a prefix delegation from an upstream device. In many cases, this means a home user obtaining a prefix delegation from the user's ISP. This is not the only application, however, and it is possible to use prefix delegation within private networks as well (a common  example would be large corporate networks).

[RFC 3633, section 5.1](https://tools.ietf.org/html/rfc3633#section-5.1) has a very good example of prefix delegation:

{{< highlight markdown>}}
5.1.  Example network architecture

   Figure 1 illustrates a network architecture in which prefix
   delegation could be used.

                 ______________________                 \
                /                      \                 \
               |    ISP core network    |                 \
                \__________ ___________/                   |
                           |                               |
                   +-------+-------+                       |
                   |  Aggregation  |                       | ISP
                   |    device     |                       | network
                   |  (delegating  |                       |
                   |    router)    |                       |
                   +-------+-------+                       |
                           |                              /
                           |DSL to subscriber            /
                           |premises                    /
                           |
                    +------+------+                     \
                    |     CPE     |                      \
                    | (requesting |                       \
                    |   router)   |                        |
                    +----+---+----+                        |
                         |   |                             | Subscriber
  ---+-------------+-----+- -+-----+-------------+---      | network
     |             |               |             |         |
+----+-----+ +-----+----+     +----+-----+ +-----+----+    |
|Subscriber| |Subscriber|     |Subscriber| |Subscriber|   /
|    PC    | |    PC    |     |    PC    | |    PC    |  /
+----------+ +----------+     +----------+ +----------+ /

   Figure 1: An example of prefix delegation.
{{< / highlight >}}

Essentially, a DHCPv6 client requests a prefix delegation in a request similar to a DHCPv6 address request. [RFC 3633, section 7](https://tools.ietf.org/html/rfc3633#section-7) specifies that these two operations can happen in the same exchange (a client can request both an address and a prefix delegation at the same time), or independently (a client can request only an address, or only a delegation). In addition, the client can specify exactly one prefix (within the range) to be excluded in the prefix delegation request, as defined in [RFC 6603](https://tools.ietf.org/html/rfc6603). The client can also include a 'hints' for preferred prefixes in the IA_PD prefix option as part of the request to the delegating router. The delegating router ultimately decides what prefixes will be delegated, as hints are not guaranteed. Even without hints in the IA_PD option of the prefix delegation request, the IA_PD serves as a unique identifier for the requester, functioning in a manner similar to how cookies in we browsers cache a little bit of session state and help to maintain a consistent result.

In contrast, IPv4 does not support prefix delegation, and the closest alternative requires a supporting cast of protocols like BGP (Border Gateway Protocol), OSPF (Open Shortest Path First), or a similar routing protocol.

For example, in order to delegate a prefix from your ISP, you would have to first obtain a contiguous address range from your ISP. Because it is 2018, I am going to assume that BGP is the routing protocol of choice. Next, you would need to register for an AS (Autonomous System) number, so that you can advertise routes to your BGP neighbors (your ISP). Next you will need to configure advertisements to your BGP neighbors, and your BGP neighbors (again your ISP), will need to configure their BGP devices to accept advertisements from you and also publish advertisements to you.


## Multicast Instead of Broadcast

Another big efficiency gain is found in the native multicast support in IPv6  and the 'Flow label' portion of the header. At the time IPv4 was defined, multicast did not exist, so many supporting protocols like [ARP (address resolution protocol)](https://tools.ietf.org/html/rfc826), rely on broadcast. This is a significant source of difficulty in IPv4 networks, since broadcast traffic must be processed by every host, even if the processing is simply reading the packet and discovering that it can be discarded, it is still unnecessary work for every host within the connected subnet. In addition to this, you must forfeit an address within each IPv4 subnet for the broadcast address.

While IPv4 *does* have a multicast implementation, it was added after the protocol was designed and is used mostly for multimedia.

In contrast, IPv6 supports multicast natively, and it is crucial to supporting protocols that make IPv6 function. The [NDP (neighbor discovery protocol)](https://tools.ietf.org/html/rfc4861) relies on multicast. [Router Advertisements and solicitation](https://tools.ietf.org/html/rfc4862#section-5.5.1) rely on multicast. This means that both SLAAC and DHCPv6 also rely on multicast. With all of these important parts of IPv6 depending on multicast, it's important to understand how it is an improvement over the IPv4 broadcast model.

[RFC 3315, section 5](https://tools.ietf.org/html/rfc3315#section-5), describes multicast groups used by DHCPv6. In [RFC 3315, section 5.3](https://tools.ietf.org/html/rfc3315#section-5.3), the types of messages seen in this multicast group. The messages are essentially a superset of similar messages used in IPv4 DHCP. [RFC 2131, page 15](https://tools.ietf.org/html/rfc2131#page-15), has an excellent overview of what the exchange between DHCP client and server look like in IPv4:

{{< highlight markdown>}}

                Server          Client          Server
            (not selected)                    (selected)

                  v               v               v
                  |               |               |
                  |     Begins initialization     |
                  |               |               |
                  | _____________/|\____________  |
                  |/DHCPDISCOVER | DHCPDISCOVER  \|
                  |               |               |
              Determines          |          Determines
             configuration        |         configuration
                  |               |               |
                  |\             |  ____________/ |
                  | \________    | /DHCPOFFER     |
                  | DHCPOFFER\   |/               |
                  |           \  |                |
                  |       Collects replies        |
                  |             \|                |
                  |     Selects configuration     |
                  |               |               |
                  | _____________/|\____________  |
                  |/ DHCPREQUEST  |  DHCPREQUEST\ |
                  |               |               |
                  |               |     Commits configuration
                  |               |               |
                  |               | _____________/|
                  |               |/ DHCPACK      |
                  |               |               |
                  |    Initialization complete    |
                  |               |               |
                  .               .               .
                  .               .               .
                  |               |               |
                  |      Graceful shutdown        |
                  |               |               |
                  |               |\ ____________ |
                  |               | DHCPRELEASE  \|
                  |               |               |
                  |               |        Discards lease
                  |               |               |
                  v               v               v
     Figure 3: Timeline diagram of messages exchanged between DHCP
               client and servers when allocating a new network address
{{< / highlight >}}

In this comparison, it is crucial to understand that the DHCPDISCOVER sent in IPv4 is a broadcast, and must be processed by every host on the subnet (even if only to discard). This is where IPv6 and multicast really shine. Because IPv6 hosts use multicast for DHCPv6, only hosts which have subscribed to the multicast group need to process DHCPv6 related messages. The process for a client obtaining an address is still very similar, but other devices on the segment do not need to handle the broadcast traffic of noisy neighbors.

With IPv6, a client requesting an address happens this way:

* The client sends a multicast Solicit message to locate a DHCPv6 server and request a leased address. This is basically a more efficient DHCPDISCOVER.

* Any server that can fulfill the client's request responds with an Advertise message. This is basically a more efficient DHCPOFFER. It's also noteworthy that multiple IPv6 DHCPv6 servers can server the same segment; with IPv4, multiple DHCP servers on a single segment is very problematic because of the required broadcast traffic.

* The client chooses one of the servers and sends a Request message, asking to confirm the address and other options from the Advertise message. This is essentially the same as a DHCPREQUEST in IPv4 DHCP.

* The DHCPv6 server responds with a Reply message to finalize the process and confirm that the client has obtained a lease on the address. This is essentially the same as a DHCPACK.

In addition, DHCPv6 also supports a "rapid commit" option, which is briefly discussed in [RFC 3315, section 22.14](https://tools.ietf.org/html/rfc3315#section-22.14). To be fair, there is an equivalent in DHCPv4 as well, which is defined in [RFC 4039](https://tools.ietf.org/html/rfc4039). I have never seen DHCPv4 rapid commit in the wild; it's possible I have obtained a DHCPv4 rapid commit lease and just didn't know, since I mostly look at DHCP when it is not doing what I expect or when I am setting up or changing a DHCP server.

Whether using normal commit mode (4 message exchange) or rapid commit mode (the 2 message exchange), IPv6 and DHCPv6 is more efficient than IPv4 and DHCPv4, through the use of multicasting rather than broadcasting.

Even the address scope reserved for multicast was chosen with efficiency in mind. The prefix for multicast addresses is `FF00::0/8`, which begins with 8 bits set to 1.

[RFC 4291, section 2](https://tools.ietf.org/html/rfc4291#section-2) explains the role of multicast in IPv6 in the very declarative terms that make RFCs such a terrific, authoritative resource:

{{< highlight markdown>}}
   Multicast: An identifier for a set of interfaces (typically
               belonging to different nodes).  A packet sent to a
               multicast address is delivered to all interfaces
               identified by that address.

   There are no broadcast addresses in IPv6, their function being
   superseded by multicast addresses.
{{< / highlight >}}

## Summary

IPv6 is conceptually very similar to IPv4 with some important differences and improvements. IPv6 is a protocol specification developed with the benefit of hindsight and decades of rigorous testing and use of IPv4. I believe that IPv6 is in many ways a superior implementation, not simply "a lot more address space". I believe the increased address space is only a fraction of what IPv6 has to offer, and to dismiss IPv6 as "too hard" or "nothing new" is a flippant and careless approach that has no place in computing. IPv6 is worth investing your time to understand and use. The topics discussed here are also only a glimpse of what the new standard has to offer, and I think as time marches forward and IPv6 adoption increases, we will see people begin to appreciate what IPv6 has to offer, and that it will enable better communication and innovation that will touch all of our lives, even for those who don't care at all about how the packets move.

### Bibliography

Links in order of appearance:

[COMPSCI X433 Fundamentals of Data Communications and Networking | UC Berkeley ExtensionUC Berkeley Extension - https://extension.berkeley.edu/search/publicCourseSearchDetails.do?method=load&courseId=40657#courseSectionDetails_43469550](https://extension.berkeley.edu/search/publicCourseSearchDetails.do?method=load&courseId=40657#courseSectionDetails_43469550)

[RFC 4291 - IP Version 6 Addressing Architecture - https://tools.ietf.org/html/rfc4291](https://tools.ietf.org/html/rfc4291)

[RFC 3022 - Traditional IP Network Address Translator (Traditional NAT) - https://tools.ietf.org/html/rfc3022](https://tools.ietf.org/html/rfc3022)

[RFC 2663 - IP Network Address Translator (NAT) Terminology and Considerations - https://tools.ietf.org/html/rfc2663](https://tools.ietf.org/html/rfc2663)

[RFC 4291 - IP Version 6 Addressing Architecture - https://tools.ietf.org/html/rfc4291#section-2.5.4](https://tools.ietf.org/html/rfc4291#section-2.5.4)

[RFC 791 - Internet Protocol - https://tools.ietf.org/html/rfc791#section-3.1](https://tools.ietf.org/html/rfc791#section-3.1)

[RFC 8200 - Internet Protocol, Version 6 (IPv6) Specification - https://tools.ietf.org/html/rfc8200#section-3](https://tools.ietf.org/html/rfc8200#section-3)

[RIPE NCC Roundtable Meeting , 16 February 2009, Amsterdam - https://www.ripe.net/participate/meetings/roundtable/february-2009/RobBlokzijlroundtable2009Rob2.pdf](https://www.ripe.net/participate/meetings/roundtable/february-2009/RobBlokzijlroundtable2009Rob2.pdf)

[Internetworking Basics - https://www.cisco.com/cpress/cc/td/cpress/fund/ith/ith01gb.htm#xtocid1668412](https://www.cisco.com/cpress/cc/td/cpress/fund/ith/ith01gb.htm#xtocid1668412)

[RFC 4291 - IP Version 6 Addressing Architecture - https://tools.ietf.org/html/rfc4291#section-2.5.6](https://tools.ietf.org/html/rfc4291#section-2.5.6)

[RFC 3927 - Dynamic Configuration of IPv4 Link-Local Addresses - https://tools.ietf.org/html/rfc3927](https://tools.ietf.org/html/rfc3927)

[RFC 4862 - IPv6 Stateless Address Autoconfiguration - https://tools.ietf.org/html/rfc4862](https://tools.ietf.org/html/rfc4862)

[RFC 4862 - IPv6 Stateless Address Autoconfiguration - https://tools.ietf.org/html/rfc4862#section-5.5.1](https://tools.ietf.org/html/rfc4862#section-5.5.1)

[RFC 4862 - IPv6 Stateless Address Autoconfiguration - https://tools.ietf.org/html/rfc4862#section-5.4](https://tools.ietf.org/html/rfc4862#section-5.4)

[RFC 4862 - IPv6 Stateless Address Autoconfiguration - https://tools.ietf.org/html/rfc4862#section-5.4.2](https://tools.ietf.org/html/rfc4862#section-5.4.2)

[RFC 4861 - Neighbor Discovery for IP version 6 (IPv6) - https://tools.ietf.org/html/rfc4861](https://tools.ietf.org/html/rfc4861)

[RFC 826 - Ethernet Address Resolution Protocol: Or Converting Network Protocol Addresses to 48.bit Ethernet Address for Transmission on Ethernet Hardware - https://tools.ietf.org/html/rfc826](https://tools.ietf.org/html/rfc826)

[RFC 3633 - IPv6 Prefix Options for Dynamic Host Configuration Protocol (DHCP) version 6 - https://tools.ietf.org/html/rfc3633#section-5.1](https://tools.ietf.org/html/rfc3633#section-5.1)

[RFC 3633 - IPv6 Prefix Options for Dynamic Host Configuration Protocol (DHCP) version 6 - https://tools.ietf.org/html/rfc3633#section-7](https://tools.ietf.org/html/rfc3633#section-7)

[RFC 6603 - Prefix Exclude Option for DHCPv6-based Prefix Delegation - https://tools.ietf.org/html/rfc6603](https://tools.ietf.org/html/rfc6603)

[RFC 826 - Ethernet Address Resolution Protocol: Or Converting Network Protocol Addresses to 48.bit Ethernet Address for Transmission on Ethernet Hardware - https://tools.ietf.org/html/rfc826](https://tools.ietf.org/html/rfc826)

[RFC 4861 - Neighbor Discovery for IP version 6 (IPv6) - https://tools.ietf.org/html/rfc4861](https://tools.ietf.org/html/rfc4861)

[RFC 4862 - IPv6 Stateless Address Autoconfiguration - https://tools.ietf.org/html/rfc4862#section-5.5.1](https://tools.ietf.org/html/rfc4862#section-5.5.1)

[RFC 3315 - Dynamic Host Configuration Protocol for IPv6 (DHCPv6) - https://tools.ietf.org/html/rfc3315#section-5](https://tools.ietf.org/html/rfc3315#section-5)

[RFC 3315 - Dynamic Host Configuration Protocol for IPv6 (DHCPv6) - https://tools.ietf.org/html/rfc3315#section-5.3](https://tools.ietf.org/html/rfc3315#section-5.3)

[RFC 2131 - Dynamic Host Configuration Protocol - https://tools.ietf.org/html/rfc2131#page-15](https://tools.ietf.org/html/rfc2131#page-15)

[RFC 3315 - Dynamic Host Configuration Protocol for IPv6 (DHCPv6) - https://tools.ietf.org/html/rfc3315#section-22.14](https://tools.ietf.org/html/rfc3315#section-22.14)

[RFC 4039 - Rapid Commit Option for the Dynamic Host Configuration Protocol version 4 (DHCPv4) - https://tools.ietf.org/html/rfc4039](https://tools.ietf.org/html/rfc4039)

[RFC 4291 - IP Version 6 Addressing Architecture - https://tools.ietf.org/html/rfc4291#section-2](https://tools.ietf.org/html/rfc4291#section-2)

