+++
author = "Chris Suttles"
categories = ["DNS", "pcap", "wireshark", "tshark", "tcpdump", "network"]
date = 2019-05-15T09:24:20Z
description = ""
draft = false
cover = "/images/2019/05/wireshark-logo.jpg"
slug = "packet-captures-and-dns"
tags = ["DNS", "pcap", "wireshark", "tshark", "tcpdump", "network"]
title = "Packet Captures and DNS"
aliases = ["/packet-captures-and-dns/"]

+++


<h1>Overview</h1>
<p>During troubleshooting, you might find it useful to determine what is happening on the wire. While tcpdump
    is a great tool for capturing packets, it's does not offer the same level of filtering capability
    as tshark. While it may be easier to simply copy a capture file locally and use wireshark, sometimes
    restrictions prevent this approach.</p>

   <h2>Capture and Save with tcpdump</h2>
   <p>Capturing and saving to disk is my favorite way to review wire traffic. I believe it makes analysis easier, since you might be able
        to use the wireshark gui (if you are allowed to retrieve the capture from the machine), or if
        using tshark locally, you can use two pass filtering and have more flexibility in choosing filters
        for analysis. For those interested in focusing on tshark, <a href="#capture-and-save-with-tshark" >the following section</a> covers doing this same thing with tshark.
        You can skip ahead and leave tcpdump to the UNIX operating systems where it came from without missing anything.
        For those of you that see the value in tcpdump (a big plus is that it's everywhere), read on.</p>
<pre>sudo tcpdump -s0 -nnpi lo -w /tmp/lo-port-53.pcap 'port 53'</pre>
   <p><br />Let's step through this command and understand what it does.</p><pre>       <b>-s </b><u>snaplen</u>
       <b>--snapshot-length=</b><u>snaplen</u>
              Snarf  <u>snaplen</u>  bytes  of  data from each packet rather than the
              default of 262144 bytes.  Packets truncated because of a limited
              snapshot  are  indicated  in the output with ``[|<u>proto</u>]'', where
              <u>proto</u> is the name of the protocol level at which the  truncation
              has  occurred.  Note that taking larger snapshots both increases
              the amount of time it takes to process packets and, effectively,
              decreases  the amount of packet buffering.  This may cause pack‐
              ets to be lost.  You should limit <u>snaplen</u> to the smallest number
              that will capture the protocol information you're interested in.
              Setting <u>snaplen</u> to 0 sets it to the default of 262144, for back‐
              wards compatibility with recent older versions of <u>tcpdump</u>.
</pre>
<p>We use <code>-s0</code>&nbsp;to capture the full packet, regardless of tcpdump version (old versions
        used a much smaller default). It is also possible to set this much lower (i.e 512 for&nbsp;<b>most</b>        DNS traffic) to get smaller capture files; this can be very useful for long running captures
        to spot traffic patterns.</p>
<pre>
  <b>-n </b>Don't  convert  host  addresses  to  names.  This can be used to
              avoid DNS lookups.

  <b>-nn </b>Don't convert protocol and port numbers etc. to names either.
</pre>
<p><br />We use <code>-nn</code>&nbsp;to prevent hostname and port/protocol name resolution at the time
        of capture. This can happen later when reviewing the pcap; doing so at the time of capture generates
        unneeded DNS traffic, and unnecessary overhead while capturing, which can cause packets to be
        dropped instead of captured at high traffic volumes.</p>
<pre>       <b>-p</b>
   <b>--no-promiscuous-mode</b>
              <u>Don't</u>  put  the  interface into promiscuous mode.  Note that the
              interface might be in promiscuous mode for  some  other  reason;
              hence,  `-p'  cannot  be used as an abbreviation for `ether host
              {local-hw-addr} or ether broadcast'.

</pre><pre>       <b>-i </b><u>interface</u>
   <b>--interface=</b><u>interface</u>
              Listen  on <u>interface</u>.  If unspecified, <u>tcpdump</u> searches the sys‐
              tem interface list for the lowest numbered, configured up inter‐
              face  (excluding  loopback), which may turn out to be, for exam‐
              ple, ``eth0''.

              On Linux systems with 2.2 or later kernels, an  <u>interface</u>  argu‐
              ment  of  ``any'' can be used to capture packets from all inter‐
              faces.  Note that captures on the ``any''  device  will  not  be
              done in promiscuous mode.

              If  the  <b>-D </b>flag is supported, an interface number as printed by
              that flag can be used as the <u>interface</u> argument, if no interface
              on the system has that number as a name.
</pre>
   <p><br />We use <code>-i lo</code>&nbsp;to capture all the traffic traversing the loopback adapter.
        This gets both bind and authcore in authoritative, because the data path goes through addresses
        bound to the loopback adapter (thanks Docker!). It can also be useful to use <code>-i any</code>,
        but use caution as this can generate a very large capture file very quickly. A good rule of thumb
        is to capture as little traffic as possible to achieve your goal.</p>
   <pre>       <b>-w </b><u>file</u>
              Write the raw packets to <u>file</u> rather than parsing  and  printing
              them  out.  They can later be printed with the -r option.  Stan‐
              dard output is used if <u>file</u> is ``-''.
</pre>
<p>We use <code>-w&nbsp;/tmp/lo-port-53.pcap</code>&nbsp;in this example to write the pcap file
 to the large file system we found in step 1, with a descriptive name of what it contains.</p>

<p>The&nbsp;'port 53' argument is a <a href="http://biot.com/capstats/bpf.html">BPF filter</a>. Here's
        a great <a href="https://blog.cloudflare.com/bpf-the-forgotten-bytecode/">write up on the underpinnings of BPF bytecode</a>,&nbsp;and
        an <a href="https://danielmiessler.com/study/tcpdump/">excellent write up on tcpdump filtering (and general use) by the venerable Daniel Miessler</a>.&nbsp;This
        is the simplest style of BPF filter possible, which says we want all traffic
        on 'port 53' (<a href="https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml?search=Domain+Name+Server">the default port for Domain Name Server as defined by IANA</a>).
        See also:&nbsp;<code>man pcap-filter</code></p>
   <h2 id="capture-and-save-with-tshark">Capture and Save with tshark</h2>
   <p>This section covers saving a capture to disk with tshark.</p>
   <pre>sudo tshark -s0 -npi lo -w /tmp/lo-port-53.pcap port 53</pre>

   <p>Let's step through this command and understand what it does.</p>
<pre>       -s  &lt;capture snaplen&gt;
           Set the default snapshot length to use when capturing live data.
           No more than <u>snaplen</u> bytes of each network packet will be read into
           memory, or saved to disk.  A value of 0 specifies a snapshot length
           of 262144, so that the full packet is captured; this is the
           default.
</pre>
   <p><br />We use&nbsp;<code>-s0</code>&nbsp;to capture the full packet, regardless of tshark version
        (old versions used a much smaller default). It is also possible to set this much lower (i.e 512
        for&nbsp; <b>most</b>&nbsp;DNS traffic) to get smaller capture files; this can be very
        useful for long running captures to spot traffic patterns. This can be omitted instead of using
        <code>-s0</code>, since in more recent versions this is the default behavior.</p>
   <pre>       -n  Disable network object name resolution (such as hostname, TCP and
           UDP port names); the <b>-N </b>option might override this one.
</pre>
   <p><span style="letter-spacing: 0.0px;">We use&nbsp;</span><code style="letter-spacing: 0.0px;">-n</code>        <span style="letter-spacing: 0.0px;">&nbsp;to prevent hostname and port/protocol name resolution at the time of capture. This can happen later when reviewing the pcap; doing so at the time of capture generates unneeded DNS traffic, and unnecessary overhead while capturing, which can cause packets to be dropped instead of captured at high traffic volumes.</span></p>
   <p><span style="letter-spacing: 0.0px;"><br /></span></p><pre>       -p  <u>Don't</u> put the interface into promiscuous mode.  Note that the
           interface might be in promiscuous mode for some other reason;
           hence, <b>-p </b>cannot be used to ensure that the only traffic that is
           captured is traffic sent to or from the machine on which <b>TShark </b>is
           running, broadcast traffic, and multicast traffic to addresses
           received by that machine.
</pre>
   <p>We prevent promiscuous mode, as not all cards support it, and we only need traffic that is actually
        addressed to us.</p>
<pre>       -i  &lt;capture interface&gt; | -
</pre>

   <p><span style="letter-spacing: 0.0px;">We use&nbsp;</span><code style="letter-spacing: 0.0px;">-i lo</code>        <span style="letter-spacing: 0.0px;">&nbsp;to capture all the traffic traversing the loopback adapter. It can also be useful to use&nbsp;</span>        <code style="letter-spacing: 0.0px;">-i any</code> <span style="letter-spacing: 0.0px;">, but use caution as this can generate a very large capture file very quickly. A good rule of thumb is to capture as little traffic as possible to achieve your goal.</span></p>
   <p><span style="letter-spacing: 0.0px;"><br /></span></p><pre>       -w  &lt;outfile&gt; | -
</pre>
   <p><span style="letter-spacing: 0.0px;">We use&nbsp;</span><code style="letter-spacing: 0.0px;">-w&nbsp;/tmp/lo-port-53.pcap</code>        <span style="letter-spacing: 0.0px;">&nbsp;in this example to write the pcap file to the large file system we found in step 1, with a descriptive name of what it contains. It's worth noting the </span>        <code style="letter-spacing: 0.0px;">-W</code> <span style="letter-spacing: 0.0px;">&nbsp;option here as well, which when combined with the </span>        <code style="letter-spacing: 0.0px;">-F</code> <span style="letter-spacing: 0.0px;">&nbsp;option will write the file in the specified format, like pcapng, which can include additional information, instead of the raw packets we write with </span>        <code style="letter-spacing: 0.0px;">-w</code> <span style="letter-spacing: 0.0px;">.&nbsp;<br /></span></p>
   <p><span style="letter-spacing: 0.0px;"><br /></span></p>
   <p><span style="letter-spacing: 0.0px;">The&nbsp;'port 53' argument is a&nbsp;</span><a style="letter-spacing: 0.0px;"
            href="http://biot.com/capstats/bpf.html">BPF filter</a><span style="letter-spacing: 0.0px;">. Here's a great&nbsp;</span>        <a href="https://blog.cloudflare.com/bpf-the-forgotten-bytecode/" style="letter-spacing: 0.0px;">write up on the underpinnings of BPF bytecode</a>        <span style="letter-spacing: 0.0px;">,&nbsp;and an&nbsp; </span> <a style="letter-spacing: 0.0px;"
            href="https://danielmiessler.com/study/tcpdump/">excellent write up on tcpdump filtering (and general use) by the venerable Daniel Miessler</a>        <span style="letter-spacing: 0.0px;">.&nbsp;This is the simplest style of BPF filter possible, which says we want all traffic on 'port 53' (</span>        <a style="letter-spacing: 0.0px;" href="https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml?search=Domain+Name+Server">the default port for Domain Name Server as defined by IANA</a>        <span style="letter-spacing: 0.0px;">). See also:&nbsp;</span> <code style="letter-spacing: 0.0px;">man pcap-filter</code></p>
   <h1>Analyze Traffic from a PCAP</h1>
   <p>Analysis of traffic saved in a packet capture is where tshark truly shines. With a saved pcap, you
        can use tshark to dissect traffic using all of the display filters available in wireshark. The
        wireshark site has an excellent reference for all of the available display filters, but of course
        the one we are most interested in is the <a href="https://www.wireshark.org/docs/dfref/d/dns.html">DNS display filter reference</a>.</p>
   <p>The Wireshark project also provides <a href="https://wiki.wireshark.org/SampleCaptures#Sample_Captures">a plethora of sample pcap files for analysis</a>.
        For the examples below, we will be using the following sample captures:<br /><br /></p>
   <ul>
   <li><a class="attachment" href="https://wiki.wireshark.org/SampleCaptures?action=AttachFile&amp;do=get&amp;target=dhcp-and-dyndns.pcap.gz"
                title="">dhcp-and-dyndns.pcap.gz</a> <span >&nbsp;(libpcap) A sample session of a host doing dhcp first and then dyndns.</span></li>
   <li><span ><a title="" href="https://wiki.wireshark.org/SampleCaptures?action=AttachFile&amp;do=get&amp;target=dns.cap" class="attachment">dns.cap</a><span >&nbsp;(libpcap) Various DNS lookups.</span><br
            /></span>
   </li>
   <li><span ><span ><a class="attachment" title="" href="https://wiki.wireshark.org/SampleCaptures?action=AttachFile&amp;do=get&amp;target=dns-remoteshell.pcap">dns-remoteshell.pcap</a><span >&nbsp;Watch frame 22 Ethereal detecting DNS Anomaly caused by remoteshell riding on DNS port - DNS Anomaly detection made easy by ethereal .. Anith Anand</span><br
            /></span>
   </span>
   </li>
   <li><span ><span ><span ><a title="" href="https://wiki.wireshark.org/SampleCaptures?action=AttachFile&amp;do=get&amp;target=dns_port.pcap" class="attachment">dns_port.pcap</a><span >&nbsp;DNS running on a different port than 53.</span><br
            /></span>
   </span>
   </span>
   </li>
   <li><span ><span ><span ><span ><a href="https://wiki.wireshark.org/SampleCaptures?action=AttachFile&amp;do=get&amp;target=dns%2Bicmp.pcapng.gz" title="" class="attachment">dns+icmp.pcapng.gz</a><span >&nbsp;DNS and ICMP saved in gzipped pcapng format.</span><br
            /></span>
   </span>
   </span>
   </span>
   </li>
   </ul>
 <h2>Simple Protocol Filtering</h2>
 <p>Here's an example capture file we can use to demonstrate filters. First let's look at the contents
        of&nbsp; <a title="" href="https://wiki.wireshark.org/SampleCaptures?action=AttachFile&amp;do=get&amp;target=dns%2Bicmp.pcapng.gz"
            class="attachment">dns+icmp.pcapng.gz</a>. One of the many features of tshark is the ability
        to automatically read captures in gzip format, so we don't need to gunzip or specify any additional
        flags for tshark to read this compressed capture directly.</p>
            <pre>tshark -r dns+icmp.pcapng.gz
    1   0.000000 192.168.43.9 → 192.168.43.1 DNS 80 Standard query 0x528e PTR 8.8.8.8.in-addr.arpa
    2   5.001009 192.168.43.9 → 192.168.43.1 DNS 80 Standard query 0x528e PTR 8.8.8.8.in-addr.arpa
    3   5.006792 192.168.43.1 → 192.168.43.9 DNS 124 Standard query response 0x528e PTR 8.8.8.8.in-addr.arpa PTR google-public-dns-a.google.com
    4   5.013334 192.168.43.9 → 8.8.8.8      ICMP 98 Echo (ping) request  id=0xd73b, seq=0/0, ttl=64
    5   5.505538      8.8.8.8 → 192.168.43.9 ICMP 98 Echo (ping) reply    id=0xd73b, seq=0/0, ttl=40 (request in 4)
    6   6.019290 192.168.43.9 → 8.8.8.8      ICMP 98 Echo (ping) request  id=0xd73b, seq=1/256, ttl=64
    7   6.153653      8.8.8.8 → 192.168.43.9 ICMP 98 Echo (ping) reply    id=0xd73b, seq=1/256, ttl=40 (request in 6)
    8   7.015108 192.168.43.9 → 8.8.8.8      ICMP 98 Echo (ping) request  id=0xd73b, seq=2/512, ttl=64
    9   7.781987      8.8.8.8 → 192.168.43.9 ICMP 98 Echo (ping) reply    id=0xd73b, seq=2/512, ttl=40 (request in 8)
   10   7.791410 192.168.43.9 → 192.168.43.1 DNS 80 Standard query 0x695d PTR 4.4.8.8.in-addr.arpa
   11   7.979359 192.168.43.1 → 192.168.43.9 DNS 124 Standard query response 0x695d PTR 4.4.8.8.in-addr.arpa PTR google-public-dns-b.google.com
   12   7.983593 192.168.43.9 → 8.8.4.4      ICMP 98 Echo (ping) request  id=0xdb3b, seq=0/0, ttl=64
   13   8.984437 192.168.43.9 → 8.8.4.4      ICMP 98 Echo (ping) request  id=0xdb3b, seq=1/256, ttl=64
   14   9.323049      8.8.4.4 → 192.168.43.9 ICMP 98 Echo (ping) reply    id=0xdb3b, seq=1/256, ttl=40 (request in 13)
   15   9.985425 192.168.43.9 → 8.8.4.4      ICMP 98 Echo (ping) request  id=0xdb3b, seq=2/512, ttl=64
   16  11.999365 192.168.43.9 → 192.168.43.1 DNS 80 Standard query 0x833a PTR 2.2.2.4.in-addr.arpa
   17  12.073341 192.168.43.1 → 192.168.43.9 DNS 116 Standard query response 0x833a PTR 2.2.2.4.in-addr.arpa PTR b.resolvers.Level3.net
   18  12.078588 192.168.43.9 → 4.2.2.2      ICMP 98 Echo (ping) request  id=0xdd3b, seq=0/0, ttl=64
   19  12.148722      4.2.2.2 → 192.168.43.9 ICMP 98 Echo (ping) reply    id=0xdd3b, seq=0/0, ttl=50 (request in 18)
   20  13.079308 192.168.43.9 → 4.2.2.2      ICMP 98 Echo (ping) request  id=0xdd3b, seq=1/256, ttl=64
   21  13.383662      4.2.2.2 → 192.168.43.9 ICMP 98 Echo (ping) reply    id=0xdd3b, seq=1/256, ttl=50 (request in 20)
   22  14.079860 192.168.43.9 → 4.2.2.2      ICMP 98 Echo (ping) request  id=0xdd3b, seq=2/512, ttl=64
   23  15.280499      4.2.2.2 → 192.168.43.9 ICMP 98 Echo (ping) reply    id=0xdd3b, seq=2/512, ttl=50 (request in 22)
   24  15.289472 192.168.43.9 → 192.168.43.1 DNS 77 Standard query 0x2121 A www.wireshark.org
   25  15.703377 192.168.43.1 → 192.168.43.9 DNS 93 Standard query response 0x2121 A www.wireshark.org A 174.137.42.65
   26  15.722009 192.168.43.9 → 192.168.43.1 DNS 77 Standard query 0x2c58 A www.wireshark.org
   27  15.865643 192.168.43.1 → 192.168.43.9 DNS 93 Standard query response 0x2c58 A www.wireshark.org A 174.137.42.65
   28  15.866126 192.168.43.9 → 174.137.42.65 ICMP 98 Echo (ping) request  id=0xe03b, seq=0/0, ttl=64
   29  16.636590 174.137.42.65 → 192.168.43.9 ICMP 98 Echo (ping) reply    id=0xe03b, seq=0/0, ttl=48 (request in 28)
   30  16.867268 192.168.43.9 → 174.137.42.65 ICMP 98 Echo (ping) request  id=0xe03b, seq=1/256, ttl=64
   31  17.194006 174.137.42.65 → 192.168.43.9 ICMP 98 Echo (ping) reply    id=0xe03b, seq=1/256, ttl=48 (request in 30)
   32  17.867597 192.168.43.9 → 174.137.42.65 ICMP 98 Echo (ping) request  id=0xe03b, seq=2/512, ttl=64
   33  18.138642 174.137.42.65 → 192.168.43.9 ICMP 98 Echo (ping) reply    id=0xe03b, seq=2/512, ttl=48 (request in 32)</pre>
   <p><br />We can see that the capture file is exactly as described; it's ICMP and DNS traffic. Let's
        pluck just the dns traffic out of this capture, using two pass filtering (-2), and using the
        dns display filter (-R dns). Note that for live captures, you can accomplish most of what can
        be done in a two-pass analysis ( <code>-2 -R</code>) with <code>-Y</code>.</p><pre>       -2  Perform a two-pass analysis. This causes tshark to buffer output
           until the entire first pass is done, but allows it to fill in
           fields that require future knowledge, such as 'response in frame #'
           fields. Also permits reassembly frame dependencies to be calculated
           correctly.
</pre><pre>       -R  &lt;Read filter&gt;
           Cause the specified filter (which uses the syntax of read/display
           filters, rather than that of capture filters) to be applied during
           the first pass of analysis. Packets not matching the filter are not
           considered for future passes. Only makes sense with multiple
           passes, see -2. For regular filtering on single-pass dissect see -Y
           instead.

           Note that forward-looking fields such as 'response in frame #'
           cannot be used with this filter, since they will not have been
           calculate when this filter is applied.

</pre><pre>       -Y  &lt;displaY filter&gt;
           Cause the specified filter (which uses the syntax of read/display
           filters, rather than that of capture filters) to be applied before
           printing a decoded form of packets or writing packets to a file.
           Packets matching the filter are printed or written to file; packets
           that the matching packets depend upon (e.g., fragments), are not
           printed but are written to file; packets not matching the filter
           nor depended upon are discarded rather than being printed or
           written.

           Use this instead of -R for filtering using single-pass analysis. If
           doing two-pass analysis (see -2) then only packets matching the
           read filter (if there is one) will be checked against this filter.

</pre>
<pre>tshark -r dns+icmp.pcapng.gz -2 -R dns
    1   0.000000 192.168.43.9 → 192.168.43.1 DNS 80 Standard query 0x528e PTR 8.8.8.8.in-addr.arpa
    2   5.001009 192.168.43.9 → 192.168.43.1 DNS 80 Standard query 0x528e PTR 8.8.8.8.in-addr.arpa
    3   5.006792 192.168.43.1 → 192.168.43.9 DNS 124 Standard query response 0x528e PTR 8.8.8.8.in-addr.arpa PTR google-public-dns-a.google.com
   10   7.791410 192.168.43.9 → 192.168.43.1 DNS 80 Standard query 0x695d PTR 4.4.8.8.in-addr.arpa
   11   7.979359 192.168.43.1 → 192.168.43.9 DNS 124 Standard query response 0x695d PTR 4.4.8.8.in-addr.arpa PTR google-public-dns-b.google.com
   16  11.999365 192.168.43.9 → 192.168.43.1 DNS 80 Standard query 0x833a PTR 2.2.2.4.in-addr.arpa
   17  12.073341 192.168.43.1 → 192.168.43.9 DNS 116 Standard query response 0x833a PTR 2.2.2.4.in-addr.arpa PTR b.resolvers.Level3.net
   24  15.289472 192.168.43.9 → 192.168.43.1 DNS 77 Standard query 0x2121 A www.wireshark.org
   25  15.703377 192.168.43.1 → 192.168.43.9 DNS 93 Standard query response 0x2121 A www.wireshark.org A 174.137.42.65
   26  15.722009 192.168.43.9 → 192.168.43.1 DNS 77 Standard query 0x2c58 A www.wireshark.org
   27  15.865643 192.168.43.1 → 192.168.43.9 DNS 93 Standard query response 0x2c58 A www.wireshark.org A 174.137.42.65
</pre>
   <h2>Managing Time</h2>
   <p>In the examples above, we see packets displayed with the default time format, which is 'relative'.
Let's look at the <code>-t</code>&nbsp;option, which allows us to specify the date/time display
format. This is very useful for correlating frames in a packet capture with logs and metrics,
or data derived from them. If you are looking at a dashboard or logs based in UTC, it
you probably want to use <code>-t u</code>&nbsp;for UTC time.</p>
<pre>
-t  a|ad|adoy|d|dd|e|r|u|ud|udoy
    Set the format of the packet timestamp printed in summary lines.
    The format can be one of:
        <b>a </b>absolute: The absolute time, as local time in your time zone, is
        the actual time the packet was captured, with no date displayed
        <b>ad </b>absolute with date: The absolute date, displayed as YYYY-MM-DD,
        and time, as local time in your time zone, is the actual time and
        date the packet was captured
        <b>adoy </b>absolute with date using day of year: The absolute date,
        displayed as YYYY/DOY, and time, as local time in your time zone,
        is the actual time and date the packet was captured
        <b>d </b>delta: The delta time is the time since the previous packet was
        captured
        <b>dd </b>delta_displayed: The delta_displayed time is the time since the
        previous displayed packet was captured
        <b>e </b>epoch: The time in seconds since epoch (Jan 1, 1970 00:00:00)
        <b>r </b>relative: The relative time is the time elapsed between the first
        packet and the current packet
        <b>u </b>UTC: The absolute time, as UTC, is the actual time the packet was
        captured, with no date displayed
        <b>ud </b>UTC with date: The absolute date, displayed as YYYY-MM-DD, and
        time, as UTC, is the actual time and date the packet was captured
        <b>udoy </b>UTC with date using day of year: The absolute date, displayed
        as YYYY/DOY, and time, as UTC, is the actual time and date the
        packet was captured

    The default format is relative.
</pre>
<h3>UTC Time</h3>
<p>Let's look at the first three lines of our previous example in UTC time with <code>-t u</code></p>
<pre>tshark -t u -r dns+icmp.pcapng.gz -2 -R dns | head -n 3
1 22:45:12.269853 192.168.43.9 → 192.168.43.1 DNS 80 Standard query 0x528e PTR 8.8.8.8.in-addr.arpa
2 22:45:17.270862 192.168.43.9 → 192.168.43.1 DNS 80 Standard query 0x528e PTR 8.8.8.8.in-addr.arpa
3 22:45:17.276645 192.168.43.1 → 192.168.43.9 DNS 124 Standard query response 0x528e PTR 8.8.8.8.in-addr.arpa PTR google-public-dns-a.google.com</pre>
<h3>Local Time</h3>
<p>Let's look at those same three example lines in local time (useful for &quot;when did I get paged?&quot;)
 with <code>-t a</code></p>
<pre>tshark -t a -r dns+icmp.pcapng.gz -2 -R dns | head -n 3
1 17:45:12.269853 192.168.43.9 → 192.168.43.1 DNS 80 Standard query 0x528e PTR 8.8.8.8.in-addr.arpa
2 17:45:17.270862 192.168.43.9 → 192.168.43.1 DNS 80 Standard query 0x528e PTR 8.8.8.8.in-addr.arpa
3 17:45:17.276645 192.168.43.1 → 192.168.43.9 DNS 124 Standard query response 0x528e PTR 8.8.8.8.in-addr.arpa PTR google-public-dns-a.google.com</pre>
<h2>Use the DNS display filter on an alternate port</h2>
<p>The default mapping of display filters/dissectors will not decode DNS traffic on an alternate
        port. Since sometimes you might need to look at DNS traffic over ports that are not the default (53), it's important
        to know how to override this behavior. We can do that by using the&nbsp;<code>-d</code>&nbsp;option.</p>
<pre>       -d  &lt;layer type&gt;==&lt;selector&gt;,&lt;decode-as protocol&gt;
       Like Wireshark's <b>Decode As... </b>feature, this lets you specify how a
       layer type should be dissected.  If the layer type in question (for
       example, <b>tcp.port </b>or <b>udp.port </b>for a TCP or UDP port number) has the
       specified selector value, packets should be dissected as the
       specified protocol.
       Example: <b>tshark -d tcp.port==8888,http </b>will decode any traffic
       running over TCP port 8888 as HTTP.
       Example: <b>tshark -d tcp.port==8888:3,http </b>will decode any traffic
       running over TCP ports 8888, 8889 or 8890 as HTTP.
       Example: <b>tshark -d tcp.port==8888-8890,http </b>will decode any traffic
       running over TCP ports 8888, 8889 or 8890 as HTTP.
       Using an invalid selector or protocol will print out a list of
       valid selectors and protocol names, respectively.
       Example: <b>tshark -d . </b>is a quick way to get a list of valid
       selectors.
       Example: <b>tshark -d ethertype==0x0800. </b>is a quick way to get a list
       of protocols that can be selected with an ethertype.
</pre>
 <p><br />Let's look at an example using the&nbsp;<a href="https://wiki.wireshark.org/SampleCaptures?action=AttachFile&amp;do=get&amp;target=dns_port.pcap"
                class="attachment" title="">dns_port.pcap</a> <span >&nbsp;sample capture.<br /><br />First, let's examine the output with no display filter, and also try to apply the filter with the default port.</span></p>
<pre>tshark -t u -r dns_port.pcap
    1 03:18:04.938672 192.168.50.50 → 192.168.0.1  UDP 75 65282 → 65333 Len=33
    2 03:18:04.945618  192.168.0.1 → 192.168.50.50 UDP 540 65333 → 65282 Len=498

tshark -t u -r dns_port.pcap -2 -R dns
`<no output>`</pre>
<p><span >Now let's tell tshark to apply the DNS display filter to our traffic so we can see what these frames contain. By specifying the UDP port in the first frame (65282) as DNS, we can correctly display the traffic using the DNS display filter.&nbsp;</span></p>
<pre>tshark -t u -d udp.port==65282,dns -r dns_port.pcap -2 -R dns
1 03:18:04.938672 192.168.50.50 → 192.168.0.1  DNS 75 Standard query 0x002b A us.pool.ntp.org
2 03:18:04.945618  192.168.0.1 → 192.168.50.50 DNS 540 Standard query response 0x002b A us.pool.ntp.org A 67.129.68.9 A 69.44.57.60 A 207.234.209.181 A 209.132.176.4 A 216.27.185.42 A 24.34.79.42 A 24.123.202.230 A 63.164.62.249 A 64.112.189.11 A 65.125.233.206 A 66.33.206.5 A 66.33.216.11 A 66.92.68.246 A 66.111.46.200 A 66.115.136.4 NS ns1.mailworx.net NS usenet.net.nz NS zbasel.fortytwo.ch NS aventura.bhms-groep.nl NS slartibartfast.bhms-groep.nl NS a.ns.madduck.net A 69.1.200.68 A 202.49.59.6</pre>
<p>Now we can see the traffic in much more detail.</p>
<h2>Getting Specific</h2>
<p>How do we look for specific DNS things in a pcap? This is where tshark, and specifically
    the power of display filters, really shines. You can accomplish a lot of the same
    things with BPF, but it is much more cumbersome and less intuitive. Let's look at
    a few examples.</p>
<h3>Find Queries by Type</h3>
<p>It's easy to filter by query type. The <a href="https://www.wireshark.org/docs/dfref/d/dns.htm">dns.qry.type filter</a>                    takes the following args and will return the query and responses that match the QType.</p>
<table
class="wrapped">
<colgroup>
    <col />
    <col />
    <col /> </colgroup>
<tbody>
    <tr>
        <td>dns.qry.type</td>
        <td>Type</td>
        <td>Unsigned integer, 2 bytes</td>
    </tr>
</tbody>
</table>
<p>The IANA DNS parameter reference is very helpful for mapping QTypes to the unsigned
    integer value (AKA the wire format):&nbsp; <a href="https://www.iana.org/assignments/dns-parameters/dns-parameters.xhtml">https://www.iana.org/assignments/dns-parameters/dns-parameters.xhtml</a></p>
<h4>A Queries</h4>
        <pre>tshark -r dhcp-and-dyndns.pcap.gz -t a -2 -R 'dns.qry.type == 1'
3 10:47:02.001067   10.20.20.4 → 10.20.20.1   DNS 79 Standard query 0x31a9 A router.far-far-away
4 10:47:02.001686   10.20.20.1 → 10.20.20.4   DNS 132 Standard query response 0x31a9 No such name A router.far-far-away SOA esel.sumpf.far-far-away
5 10:47:02.002032   10.20.20.4 → 10.20.20.1   DNS 79 Standard query 0x31aa A router.far-far-away
6 10:47:02.002529   10.20.20.1 → 10.20.20.4   DNS 132 Standard query response 0x31aa No such name A router.far-far-away SOA esel.sumpf.far-far-away
64 10:48:04.390038  10.20.20.20 → 10.20.20.1   DNS 76 Standard query 0x003c A sus.far-far-away
65 10:48:04.390503   10.20.20.1 → 10.20.20.20  DNS 129 Standard query response 0x003c No such name A sus.far-far-away SOA esel.sumpf.far-far-away
66 10:48:04.390767  10.20.20.20 → 10.20.20.1   DNS 76 Standard query 0x003d A sus.far-far-away
67 10:48:04.391024   10.20.20.1 → 10.20.20.20  DNS 129 Standard query response 0x003d No such name A sus.far-far-away SOA esel.sumpf.far-far-away</pre>
<h4>NS Queries</h4>
<pre>tshark -r dns.cap -t a -2 -R 'dns.qry.type == 2'
27 02:52:17.740166 192.168.170.8 → 192.168.170.20 DNS 67 Standard query 0x208a NS isc.org
29 02:52:17.758453 192.168.170.20 → 192.168.170.8 DNS 166 Standard query response 0x208a NS isc.org NS ns-ext.nrt1.isc.org NS ns-ext.sth1.isc.org NS ns-ext.isc.org NS ns-ext.lga1.isc.org</pre>
<p>Note this is a different pcap than the other examples in this section.</p>
<h4>SOA Queries</h4>
        <pre>tshark -r dhcp-and-dyndns.pcap.gz -t a -2 -R 'dns.qry.type == 6'
    9 10:47:02.008151   10.20.20.4 → 10.20.20.1   DNS 239 Dynamic update 0x21eb SOA far-far-away ANY A 10.20.20.20 TXT TSIG
   11 10:47:02.090492   10.20.20.4 → 10.20.20.1   DNS 214 Dynamic update 0x21ec SOA 20.20.10.in-addr.arpa PTR PTR academy04.far-far-away TSIG</pre>
<h4>TSIG Queries</h4>
            <pre>tshark -r dhcp-and-dyndns.pcap.gz -t a -2 -R 'dns.qry.type == 250'
<no output></pre>
<p>This filter is the same with any QType, so just substitute the appropriate integer
    value for the QType you want.</p>
<h3>Find Queries by Name</h3>
<p>We can use the dns.qry.name display filter to look for queries and responses by name.</p>
        <pre>tshark -r dns.cap -t a -2 -R 'dns.qry.name == isc.org'
    1 02:52:17.740166 192.168.170.8 → 192.168.170.20 DNS 67 Standard query 0x208a NS isc.org
    2 02:52:17.758453 192.168.170.20 → 192.168.170.8 DNS 166 Standard query response 0x208a NS isc.org NS ns-ext.nrt1.isc.org NS ns-ext.sth1.isc.org NS ns-ext.isc.org NS ns-ext.lga1.isc.org</pre>
    <h3>Find a Specific Frame</h3>
    <p>Sometimes you might want to pluck a specific frame or series of frames from a
        capture. You can do that with the frame filter. In the following example
        we filter for just frame 22, from the following sample capture:</p>
    <ul>
        <li><span ><a href="https://wiki.wireshark.org/SampleCaptures?action=AttachFile&amp;do=get&amp;target=dns-remoteshell.pcap" class="attachment" title="">dns-remoteshell.pcap</a><span>&nbsp;Watch frame 22 Ethereal detecting DNS Anomaly caused by remoteshell riding on DNS port - DNS Anomaly detection made easy by ethereal .. Anith Anand</span><br
            /></span>
        </li>
    </ul>
                <pre>tshark -r dns-remoteshell.pcap -t a -2 -R 'frame.number == 22'
   22 19:50:42.040365  192.168.1.2 → 192.168.1.3  TCP 142 [TCP Retransmission] 53 → 1396 [PSH, ACK] Seq=1 Ack=1 Win=65535 Len=88</pre>
    <p>Here, we are looking at the specific frame mentioned as a remote shell masquerading
        as DNS port. This view doesn't tell us much besides that the DNS display
        filter didn't find valid information to decode. If you look at the output
        of the following command, filtering with just the DNS display filter, we
        can see that this frame is omitted:</p>
            <pre>tshark -r dns-remoteshell.pcap -t a -2 -R dns
    2 19:50:16.501471  192.168.1.3 → 192.168.1.1  DNS 84 Standard query 0x0001 PTR 1.1.168.192.in-addr.arpa
    4 19:50:16.504144  192.168.1.1 → 192.168.1.3  DNS 112 Standard query response 0x0001 PTR 1.1.168.192.in-addr.arpa PTR SpeedTouch.lan
    7 19:50:19.113417  192.168.1.3 → 192.168.1.1  DNS 75 Standard query 0x0002 A www.www.com.lan
    8 19:50:19.131199  192.168.1.1 → 192.168.1.3  DNS 75 Standard query response 0x0002 A www.www.com.lan
    9 19:50:19.132818  192.168.1.3 → 192.168.1.1  DNS 71 Standard query 0x0003 A www.www.com
   10 19:50:19.442482  192.168.1.1 → 192.168.1.3  DNS 87 Standard query response 0x0003 A www.www.com A 63.215.91.200</pre>
    <p>To see what is actually in this frame, which confirms it as a remote shell, look
        at&nbsp;<a href="#dump-hex-and-ascii">&quot;Dump Packet Hex and ASCII&quot;</a></p>
    <h3>Find Responses by Type</h3>
            <pre>tshark -t u -r dns.cap -2 -R 'dns.resp.type == 1'
    1 08:47:51.333401 192.168.170.20 → 192.168.170.8 DNS 298 Standard query response 0xf76f MX google.com MX 40 smtp4.google.com MX 10 smtp5.google.com MX 10 smtp6.google.com MX 10 smtp1.google.com MX 10 smtp2.google.com MX 40 smtp3.google.com A 216.239.37.26 A 64.233.167.25 A 66.102.9.25 A 216.239.57.25 A 216.239.37.25 A 216.239.57.26
    2 08:49:18.734862 192.168.170.20 → 192.168.170.8 DNS 90 Standard query response 0x75c0 A www.netbsd.org A 204.152.190.12
    3 08:52:17.733384 192.168.170.20 → 192.168.170.8 DNS 115 Standard query response 0xfee3 ANY www.isc.org AAAA 2001:4f8:0:2::d A 204.152.184.88</pre>
    <h3>Find Responses by Time (length of time to respond)</h3>
    <p>This is most useful when combined with other options, so that the dns.time is
        actually displayed. See &quot;Displaying Specific Fields&quot;
        for an example.</p>
            <pre>tshark -r dns.cap -t a -2 -R 'dns.time > .06'
    1 02:47:51.333401 192.168.170.20 → 192.168.170.8 DNS 298 Standard query response 0xf76f MX google.com MX 40 smtp4.google.com MX 10 smtp5.google.com MX 10 smtp6.google.com MX 10 smtp1.google.com MX 10 smtp2.google.com MX 40 smtp3.google.com A 216.239.37.26 A 64.233.167.25 A 66.102.9.25 A 216.239.57.25 A 216.239.37.25 A 216.239.57.26
    2 02:47:59.452255 192.168.170.20 → 192.168.170.8 DNS 70 Standard query response 0x49a1 LOC google.com
    3 02:49:35.698849 192.168.170.20 → 192.168.170.8 DNS 102 Standard query response 0xf0d4 AAAA www.netbsd.org AAAA 2001:4f8:4:7:2e0:81ff:fe52:9a6b
    4 02:51:35.437491 192.168.170.20 → 192.168.170.8 DNS 75 Standard query response 0xbc1f AAAA www.example.com
    5 02:51:47.032976 192.168.170.20 → 192.168.170.8 DNS 79 Standard query response 0x266d No such name AAAA www.example.notginh
    6 02:52:17.733384 192.168.170.20 → 192.168.170.8 DNS 115 Standard query response 0xfee3 ANY www.isc.org AAAA 2001:4f8:0:2::d A 204.152.184.88</pre>
    <h3>Find Responses by Name</h3>
            <pre>tshark -r dns.cap -t a -2 -R 'dns.resp.name == google.com'
    1 02:47:46.496576 192.168.170.20 → 192.168.170.8 DNS 98 Standard query response 0x1032 TXT google.com TXT
    2 02:47:51.333401 192.168.170.20 → 192.168.170.8 DNS 298 Standard query response 0xf76f MX google.com MX 40 smtp4.google.com MX 10 smtp5.google.com MX 10 smtp6.google.com MX 10 smtp1.google.com MX 10 smtp2.google.com MX 40 smtp3.google.com A 216.239.37.26 A 64.233.167.25 A 66.102.9.25 A 216.239.57.25 A 216.239.37.25 A 216.239.57.26</pre>
    <h3>Getting More Details</h3>
    <p><span >So far we've been looking at traffic at a pretty high level, but what there's a couple of ways we can get more detail.</span></p>
    <h3><span >Display Filter Decode</span></h3>
    <p><span >The first, and easiest way it to leverage the -O option.</span></p>
    <pre>       -O  &lt;protocols&gt;
           Similar to the <b>-V </b>option, but causes <b>TShark </b>to only show a detailed
           view of the comma-separated list of <u>protocols</u> specified, and show
           only the top-level detail line for all other protocols, rather than
           a detailed view of all protocols.  Use the output of &quot;<b>tshark -G</b>
           <b>protocols</b>&quot; to find the abbreviations of the protocols you can
           specify.

</pre><pre>       -V  Cause <b>TShark </b>to print a view of the packet details.
</pre>
    <p><span ><br /></span></p>
    <p><span >The -V option is included for reference/completeness sake, but it is typically too verbose for tracing a protocol like DNS and is more useful when you want to look at lower level things (Ethernet/TCP), or do parsing with grep/awk/perl etc.<br /></span></p>
            <pre>tshark -r dns.cap -t a -2 -R 'dns.qry.type == 2' -O dns
Frame 27: 67 bytes on wire (536 bits), 67 bytes captured (536 bits)
Ethernet II, Src: AsustekI_b1:0c:ad (00:e0:18:b1:0c:ad), Dst: QuantaCo_32:41:8c (00:c0:9f:32:41:8c)
Internet Protocol Version 4, Src: 192.168.170.8, Dst: 192.168.170.20
User Datagram Protocol, Src Port: 32797, Dst Port: 53
Domain Name System (query)
    Transaction ID: 0x208a
    Flags: 0x0100 Standard query
        0... .... .... .... = Response: Message is a query
        .000 0... .... .... = Opcode: Standard query (0)
        .... ..0. .... .... = Truncated: Message is not truncated
        .... ...1 .... .... = Recursion desired: Do query recursively
        .... .... .0.. .... = Z: reserved (0)
        .... .... ...0 .... = Non-authenticated data: Unacceptable
    Questions: 1
    Answer RRs: 0
    Authority RRs: 0
    Additional RRs: 0
    Queries
        isc.org: type NS, class IN
            Name: isc.org
            [Name Length: 7]
            [Label Count: 2]
            Type: NS (authoritative Name Server) (2)
            Class: IN (0x0001)
    [Response In: 29]

Frame 29: 166 bytes on wire (1328 bits), 166 bytes captured (1328 bits)
Ethernet II, Src: QuantaCo_32:41:8c (00:c0:9f:32:41:8c), Dst: AsustekI_b1:0c:ad (00:e0:18:b1:0c:ad)
Internet Protocol Version 4, Src: 192.168.170.20, Dst: 192.168.170.8
User Datagram Protocol, Src Port: 53, Dst Port: 32797
Domain Name System (response)
    Transaction ID: 0x208a
    Flags: 0x8180 Standard query response, No error
        1... .... .... .... = Response: Message is a response
        .000 0... .... .... = Opcode: Standard query (0)
        .... .0.. .... .... = Authoritative: Server is not an authority for domain
        .... ..0. .... .... = Truncated: Message is not truncated
        .... ...1 .... .... = Recursion desired: Do query recursively
        .... .... 1... .... = Recursion available: Server can do recursive queries
        .... .... .0.. .... = Z: reserved (0)
        .... .... ..0. .... = Answer authenticated: Answer/authority portion was not authenticated by the server
        .... .... ...0 .... = Non-authenticated data: Unacceptable
        .... .... .... 0000 = Reply code: No error (0)
    Questions: 1
    Answer RRs: 4
    Authority RRs: 0
    Additional RRs: 0
    Queries
        isc.org: type NS, class IN
            Name: isc.org
            [Name Length: 7]
            [Label Count: 2]
            Type: NS (authoritative Name Server) (2)
            Class: IN (0x0001)
    Answers
        isc.org: type NS, class IN, ns ns-ext.nrt1.isc.org
            Name: isc.org
            Type: NS (authoritative Name Server) (2)
            Class: IN (0x0001)
            Time to live: 3600
            Data length: 14
            Name Server: ns-ext.nrt1.isc.org
        isc.org: type NS, class IN, ns ns-ext.sth1.isc.org
            Name: isc.org
            Type: NS (authoritative Name Server) (2)
            Class: IN (0x0001)
            Time to live: 3600
            Data length: 14
            Name Server: ns-ext.sth1.isc.org
        isc.org: type NS, class IN, ns ns-ext.isc.org
            Name: isc.org
            Type: NS (authoritative Name Server) (2)
            Class: IN (0x0001)
            Time to live: 3600
            Data length: 9
            Name Server: ns-ext.isc.org
        isc.org: type NS, class IN, ns ns-ext.lga1.isc.org
            Name: isc.org
            Type: NS (authoritative Name Server) (2)
            Class: IN (0x0001)
            Time to live: 3600
            Data length: 14
            Name Server: ns-ext.lga1.isc.org
    [Request In: 27]
    [Time: 0.018287000 seconds]
</pre>
                            <p><span >This view can be useful if you want to inspect the flags or TTL of a specific query (or set of queries), or other information not displayed in the default, summary view.</span></p>
                            <h3 id="dump-hex-and-ascii">Dump Packet Hex and ASCII</h3>
                            <p><span >To dump the contents of a frame in hex and ASCII, we can use the <code>-x</code>&nbsp;option.<br /></span></p>
<pre>
       -x  Cause <b>TShark </b>to print a hex and ASCII dump of the packet data after
           printing the summary and/or details, if either are also being
           displayed.

</pre>
<pre>tshark -r dns-remoteshell.pcap -t a -2 -R 'frame.number == 22' -x
0000  00 0e 35 78 0c 02 00 80 48 24 33 32 08 00 45 00   ..5x....H$32..E.
0010  00 80 07 cd 40 00 40 06 af 55 c0 a8 01 02 c0 a8   ....@.@..U......
0020  01 03 00 35 05 74 bd 0f 2f ed 23 c5 33 c0 50 18   ...5.t../.#.3.P.
0030  ff ff 4e 14 00 00 4d 69 63 72 6f 73 6f 66 74 20   ..N...Microsoft
0040  57 69 6e 64 6f 77 73 20 58 50 20 5b 56 65 72 73   Windows XP [Vers
0050  69 6f 6e 20 35 2e 31 2e 32 36 30 30 5d 0d 0a 28   ion 5.1.2600]..(
0060  43 29 20 43 6f 70 79 72 69 67 68 74 20 31 39 38   C) Copyright 198
0070  35 2d 32 30 30 31 20 4d 69 63 72 6f 73 6f 66 74   5-2001 Microsoft
0080  20 43 6f 72 70 2e 0d 0a 0d 0a 43 3a 5c 3e          Corp.....C:\&gt;</pre>
<p><span >Here we can see the confirmation that even though this frame is TCP over port 53, it is&nbsp;<b>not</b> DNS (as indicated by the text describing the sample capture).<br /></span></p>
<p><span >For the sake of comparison, here's a hex/ASCII decode of a pair of real DNS frames. The first command is selecting the dns.qry.type 1 and frames numbered less than 11. The second command is dumping the hex/ASCII decodes of both frames.<br /></span></p>
<pre>tshark -r dns.cap -t a -2 -R 'dns.qry.type == 1 and frame.number &lt; 11'
    9 02:49:18.685951 192.168.170.8 → 192.168.170.20 DNS 74 Standard query 0x75c0 A www.netbsd.org
   10 02:49:18.734862 192.168.170.20 → 192.168.170.8 DNS 90 Standard query response 0x75c0 A www.netbsd.org A 204.152.190.12
</pre>
<pre>
tshark -r dns.cap -t a -2 -R 'dns.qry.type == 1 and frame.number &lt; 11' -x
0000  00 c0 9f 32 41 8c 00 e0 18 b1 0c ad 08 00 45 00   ...2A.........E.
0010  00 3c 00 00 40 00 40 11 65 43 c0 a8 aa 08 c0 a8   .&lt;..@.@.eC......
0020  aa 14 80 1b 00 35 00 28 af 61 75 c0 01 00 00 01   .....5.(.au.....
0030  00 00 00 00 00 00 03 77 77 77 06 6e 65 74 62 73   .......www.netbs
0040  64 03 6f 72 67 00 00 01 00 01                     d.org.....

0000  00 e0 18 b1 0c ad 00 c0 9f 32 41 8c 08 00 45 00   .........2A...E.
0010  00 4c cf f9 00 00 80 11 95 39 c0 a8 aa 14 c0 a8   .L.......9......
0020  aa 08 00 35 80 1b 00 38 a3 17 75 c0 81 80 00 01   ...5...8..u.....
0030  00 01 00 00 00 00 03 77 77 77 06 6e 65 74 62 73   .......www.netbs
0040  64 03 6f 72 67 00 00 01 00 01 c0 0c 00 01 00 01   d.org...........
0050  00 01 40 ef 00 04 cc 98 be 0c                     ..@.......
</pre>
<h3>Displaying Specific Fields</h3>
<p>Sometimes, you want to look at just a specific field or fields within
    DNS traffic, and viewing details as provided by the -O option is
    too verbose. This is where the combination of options -T and -e/-E
    are very useful. The -T option controls the output format and is
    very flexible. The -e and -E options control what fields and how
    to format them.</p>
{{< rawhtml >}}
<pre>
       -T  ek|fields|json|jsonraw|pdml|ps|psml|tabs|text
           Set the format of the output when viewing decoded packet data.  The
           options are one of:

           <b>ek </b>Newline delimited JSON format for bulk import into
           Elasticsearch.  It can be used with <b>-j </b>or <b>-J </b>including the JSON
           filter or with <b>-x </b>to include raw hex-encoded packet data.  If <b>-P </b>is
           specified it will print the packet summary only, with both <b>-P </b>and
           <b>-V </b>it will print the packet summary and packet details.  If neither
           <b>-P </b>or <b>-V </b>are used it will print the packet details only.  Example
           of usage to import data into Elasticsearch:

             tshark -T ek -j &quot;http tcp ip&quot; -P -V -x -r file.pcap &gt; file.json
             curl -H &quot;Content-Type: application/x-ndjson&quot; -XPOST http://elasticsearch:9200/_bulk --data-binary &quot;@file.json&quot;

           Elastic requires a mapping file to be loaded as template for
           packets-* index in order to convert wireshark types to elastic
           types. This file can be auto-generated with the command &quot;tshark -G
           elastic-mapping&quot;. Since the mapping file can be huge, protocols can
           be selected by using the option --elastic-mapping-filter:

             tshark -G elastic-mapping --elastic-mapping-filter ip,udp,dns

           <b>fields </b>The values of fields specified with the <b>-e </b>option, in a form
           specified by the <b>-E </b>option.  For example,

             tshark -T fields -E separator=, -E quote=d

           would generate comma-separated values (CSV) output suitable for
           importing into your favorite spreadsheet program.

           <b>json </b>JSON file format.  It can be used with <b>-j </b>or <b>-J </b>including the
           JSON filter or with <b>-x </b>option to include raw hex-encoded packet
           data.  Example of usage:
             tshark -T jsonraw -r file.pcap
             tshark -T jsonraw -j &quot;http tcp ip&quot; -x -r file.pcap

           <b>pdml </b>Packet Details Markup Language, an XML-based format for the
           details of a decoded packet.  This information is equivalent to the
           packet details printed with the <b>-V </b>option.  Using the --color
           option will add color attributes to <b>pdml </b>output.  These attributes
           are nonstandard.

           <b>ps </b>PostScript for a human-readable one-line summary of each of the
           packets, or a multi-line view of the details of each of the
           packets, depending on whether the <b>-V </b>option was specified.

           <b>psml </b>Packet Summary Markup Language, an XML-based format for the
           summary information of a decoded packet.  This information is
           equivalent to the information shown in the one-line summary printed
           by default.  Using the --color option will add color attributes to
           <b>pdml </b>output. These attributes are nonstandard.

           <b>tabs </b>Similar to the default <b>text </b>report except the human-readable
           one-line summary of each packet will include an ASCII horizontal
           tab (0x09) character as a delimiter between each column.

           <b>text </b>Text of a human-readable one-line summary of each of the
           packets, or a multi-line view of the details of each of the
           packets, depending on whether the <b>-V </b>option was specified.  This is
           the default.

       -e  &lt;field&gt;
           Add a field to the list of fields to display if <b>-T</b>
           <b>ek|fields|json|pdml </b>is selected.  This option can be used multiple
           times on the command line.  At least one field must be provided if
           the <b>-T fields </b>option is selected. Column names may be used prefixed
           with &quot;_ws.col.&quot;

           Example: <b>tshark -e frame.number -e ip.addr -e udp -e _ws.col.Info</b>

           Giving a protocol rather than a single field will print multiple
           items of data about the protocol as a single field.  Fields are
           separated by tab characters by default.  <b>-E </b>controls the format of
           the printed fields.

       -E  &lt;field print option&gt;
           Set an option controlling the printing of fields when <b>-T fields </b>is
           selected.

           Options are:

           <b>bom=y|n </b>If <b>y</b>, prepend output with the UTF-8 byte order mark
           (hexadecimal ef, bb, bf). Defaults to <b>n</b>.

           <b>header=y|n </b>If <b>y</b>, print a list of the field names given using <b>-e </b>as
           the first line of the output; the field name will be separated
           using the same character as the field values.  Defaults to <b>n</b>.

           <b>separator=/t|/s|</b>&lt;character&gt; Set the separator character to use for
           fields.  If <b>/t </b>tab will be used (this is the default), if <b>/s</b>, a
           single space will be used.  Otherwise any character that can be
           accepted by the command line as part of the option may be used.

           <b>occurrence=f|l|a </b>Select which occurrence to use for fields that
           have multiple occurrences.  If <b>f </b>the first occurrence will be used,
           if <b>l </b>the last occurrence will be used and if <b>a </b>all occurrences will
           be used (this is the default).

           <b>aggregator=,|/s|</b>&lt;character&gt; Set the aggregator character to use for
           fields that have multiple occurrences.  If <b>, </b>a comma will be used
           (this is the default), if <b>/s</b>, a single space will be used.
           Otherwise any character that can be accepted by the command line as
           part of the option may be used.

           <b>quote=d|s|n </b>Set the quote character to use to surround fields.  <b>d</b>
           uses double-quotes, <b>s </b>single-quotes, <b>n </b>no quotes (the default).
</pre>
{{< / rawhtml >}}

<p>Let's look at some examples that are useful for DNS. We'll start by printing
fields that are pretty close to the default output, and also filter
for a specific name.</p>
<pre>tshark -e frame.number -e ip.addr -e udp -e _ws.col.Info -Tfields -r dns.cap -2 -R 'dns.qry.name == isc.org'
1       192.168.170.8,192.168.170.20    User Datagram Protocol, Src Port: 32797, Dst Port: 53   Standard query 0x208a NS isc.org
2       192.168.170.20,192.168.170.8    User Datagram Protocol, Src Port: 53, Dst Port: 32797   Standard query response 0x208a NS isc.org NS ns-ext.nrt1.isc.org NS ns-ext.sth1.isc.org NS ns-ext.isc.org NS ns-ext.lga1.isc.org</pre>
            <p>Next, let's trim things down and filter for just A type responses, and
                print only the query name and time.</p>
<pre>tshark -r dns.cap -2 -R 'dns.resp.type == 1' -T fields -e dns.qry.name -e dns.time
google.com      0.832133000
www.netbsd.org  0.048911000
www.isc.org     0.072604000</pre>
            <h1>Combining Filters into Expressions with Operators</h1>
            <p>We've covered this in a few examples, but it's so valuable that it is
                getting its own section. By now, the power of display filters for
                parsing a packet capture is very clear. When you add the logic operators
                to the mix and start chaining them together, it becomes possible
                to make very specific filters that will enable you to easily find
                the proverbial &quot;needle in a haystack&quot; within a packet capture.
                You can find <a href="https://www.wireshark.org/docs/wsug_html_chunked/ChWorkBuildDisplayFilterSection.html#FiltLogOps">the full reference for logic operators on the Wireshark website.</a></p>
            <h3>The AND Operator</h3>
            <p>In the first example we will find all AAAA queries by type, which took
                over .22 seconds for response.</p>
<pre>tshark -r dns.cap -t a -2 -R 'dns.qry.type == 28 and dns.time > .22'
    1 02:49:35.698849 192.168.170.20 → 192.168.170.8 DNS 102 Standard query response 0xf0d4 AAAA www.netbsd.org AAAA 2001:4f8:4:7:2e0:81ff:fe52:9a6b
    2 02:51:35.437491 192.168.170.20 → 192.168.170.8 DNS 75 Standard query response 0xbc1f AAAA www.example.com</pre>
            <h3>The OR Operator</h3>
            <p>Next, we will look for query types of IXFR or AXFR. There is no output
                to this command because this capture (indeed none of the sample captures)
                have IXFR or AXFR queries included.</p>
<pre>tshark -r dns.cap -t a -2 -R 'dns.qry.type == 251 or dns.qry.type == 252'
<no output></pre>
            <p>It's important to note that you could accomplish this same goal with
                multiple display filter (-R) arguments, but doing a single display
                filter with the OR operator is more readable and more efficient.</p>
            <h3>Comparison Operators</h3>
            <p>We've already been using some simple comparison operators, but you can
                find <a href="https://www.wireshark.org/docs/wsug_html_chunked/ChWorkBuildDisplayFilterSection.html#DispCompOps">the complete reference for comparison operators on the Wireshark website.</a>                                    We can build on some of our previous examples and get very specific.
                Let's add to the AND operator example. Here we will filter based
                on many criteria and operators: dns.time, dns.qry.type, and a grouped
                comparison for a time window using the absolute frame time. It's
                worth noting we could omit the parenthesis, but they are added for
                readability and explicit interpolation.</p>
<pre>tshark -r dns.cap -t a -2 -R 'dns.qry.type == 28 and dns.time > .01 and ("2005-03-30 02:51:47" > frame.time >= "2005-03-30 02:51:35")'
    1 02:51:35.437491 192.168.170.20 → 192.168.170.8 DNS 75 Standard query response 0xbc1f AAAA www.example.com</pre>
            <h3>Membership Operator</h3>
            <p>There's another powerful operator we can use to make our expressions
                very precise. The membership operator allows us to create groups
                of acceptable values and apply them to filters. Here's an example,
                building on our comparison operator example. Using the membership
                operator, we can look for both A and AAAA queries that also match
                our dns.time and frame.time criteria. Note that the frame.time is
                expressed with nanosecond precision.</p>
<pre>tshark -r dns.cap -t a -2 -R 'dns.qry.type in { 1 28 } and dns.time > .01 and ("2005-03-30 2:49:35.698851" > frame.time >= "2005-03-30 02:49:18.734861")'
    1 02:49:18.734862 192.168.170.20 → 192.168.170.8 DNS 90 Standard query response 0x75c0 A www.netbsd.org A 204.152.190.12
    2 02:49:35.698849 192.168.170.20 → 192.168.170.8 DNS 102 Standard query response 0xf0d4 AAAA www.netbsd.org AAAA 2001:4f8:4:7:2e0:81ff:fe52:9a6b</pre>
            <h1><br />Analyze Live Traffic</h1>
            <p>When analyzing live traffic, you can combine most of the filters and
                options described previously, with the following notable exceptions:
                you must use <code>-Y</code>&nbsp;in place of <code>-2 -R</code>,
                and filtering by frame number is not practical or desirable. Analyzing
                live traffic requires that you put some forethought into what you
                will capture and how, so that you get everything you need to accomplish
                your goal, without getting too much additional information and ruining
                the signal:noise ratio.
                Unless there is an urgent reason to look at live traffic, it's usually preferable to capture to disk so that you can do analysis
                later.</p>





