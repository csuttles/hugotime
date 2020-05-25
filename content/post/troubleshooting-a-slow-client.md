+++
author = "Chris Suttles"
categories = ["golang", "AWS", "IPv6", "IPv4", "network", "HTTP", "API"]
date = 2018-12-13T04:56:00Z
description = ""
draft = false
cover = "/images/2018/12/golang-debug-1.png"
slug = "troubleshooting-a-slow-client"
tags = ["golang", "AWS", "IPv6", "IPv4", "network", "HTTP", "API"]
title = "Troubleshooting a Slow API Client in Golang"
aliases = ["/troubleshooting-a-slow-client/"]

+++


I'm writing an API client in golang, and while testing the API with curl, my results were reasonable (0.8 seconds REAL time), but when I got my API client working in golang with [resty](https://github.com/go-resty/resty), I found that my response time was very slow (over 75 seconds).

This is a true (and ugly) story about figuring out what was wrong and resolving the issue.Here's my boilerplate resty code:

{{< highlight go >}}
  resty.SetDebug(true)
  resty.SetTLSClientConfig(&tls.Config{ InsecureSkipVerify: true })
  resp, err := resty.R().
    SetQueryString(fmt.Sprintf("tasks", "6")).
    SetHeader("Accept", "application/json").
    Get("https://mycoolservice.com/api/v1")
  if err != nil {
    log.Fatalf("we exploded: %s\n", err)
  }

  fmt.Printf("\nError: %v", err)
  fmt.Printf("\nResponse Status Code: %v", resp.StatusCode())
  fmt.Printf("\nResponse Status: %v", resp.Status())
  fmt.Printf("\nResponse Time: %v", resp.Time())
  fmt.Printf("\nResponse Received At: %v", resp.ReceivedAt())
  fmt.Printf("\nResponse Body: %v", resp)     // or resp.String() or string(resp.Body())
{{< / highlight>}}

I'm not doing anything tricky or special here, it's almost verbatim for the simplest use case in the docs. So what could be going on here? Why is resty taking so long?

I used `resty.SetDebug(true)` in my initial troubleshooting, but I didn't get enough detail, or just as likely failed to spot the problem before digging deeper.

While resty has a ton of features, my use case is pretty simple, so I decided to try using just the stdlib, so I wrote something to talk to the API with `net/http` and friends. Even though this next round was super simple, no-frills, I still saw ridiculous response times.

## What Is Going On?

I decided to start stepping through the program and set some breakpoints in the debugger. From here, I could see that everything else was pretty zippy until I asked the `http.Client` to get the request via `Client.do(req)`. Well that's good news and bad news at the same time. The good news is that I didn't introduce a mistake that was causing the delay, but the bad news was that I didn't understand why the request was hanging. I tried stepping into the `Client.do`, but that quickly became a rabbit hole and for me the signal:noise ratio wasn't working.Next, I tried using what I knew about the API I was talking to, and tried influencing the client behavior. The `net/http` package will try to use HTTP2 by default, so I tried disabling that, thinking that perhaps it was causing a timeout trying HTTP2 before falling back to HTTP1.1.Next, I tried setting timeouts and other Transport settings, but these also yielded no improvement.Now I took a step back and decided to get more information, since my guesses had lead nowhere. I took a look at [http-tracing](https://blog.golang.org/http-tracing) and jammed that into the code that I was using to build the client:

{{< highlight go>}}
  trace := &httptrace.ClientTrace{
    DNSDone: func(dnsInfo httptrace.DNSDoneInfo) {
      fmt.Printf("%v DNS Info: %+v\n", time.Now(), dnsInfo)
    },
    GotConn: func(connInfo httptrace.GotConnInfo) {
      fmt.Printf("%v Got Conn: %+v\n", time.Now(), connInfo)
    },
  }
  req = req.WithContext(httptrace.WithClientTrace(req.Context(), trace))

  // do the actual request and handle it
  client := &http.Client{Transport: tr,
    Timeout: time.Second * 180,
  }
{{< / highlight>}}

Now I could see where things were stalling:

{{< highlight go >}}
2018-12-10 20:14:32.395625 -0600 CST m=+0.003785955 DNS Info: {Addrs:[{IP:2001:DB8::face:cafe:d00d:c001 Zone:} {IP:169.254.169.254 Zone:}] Err:<nil> Coalesced:false}
2018-12-10 20:15:48.295515 -0600 CST m=+75.905881237 Got Conn: {Conn:0xc0000b2a80 Reused:false WasIdle:false IdleTime:0s}
{{< / highlight >}}

From the two timestamps, we can see that the connection is established 76 seconds after the DNS lookup is completed.

## Why Is This Happening?

When I looked at the the `net/http/httptrace` output, I noticed that the result of the DNS lookup was a `AAAA` record followed by an `A` record. I've had my share of fun with IPv6 in the past and thought that there is a good chance IPv6 is not set up correctly on the API I was trying to reach. I decided to test this theory with our old pal `curl`. Sure enough, I could `curl -4 $URL` very quickly, but `curl -6 $URL` was hanging until timeout.

Now I was getting somewhere and understood the cause of the delay. When I was poking this API with `curl` without specifying the `-4` or `-6` args, it was quickly falling back to IPv4 and giving me a response, but the default client parameters in the `net/http` client in golang are different, so it was trying to connect to the IPv6 it found in, and taking a long time to timeout before falling back to IPv4 and succeeding. I had tried fiddling with timeouts in `net/http`, but not the right ones.

## Fixing The API

The next step was to look at the remote side and fix it. This API lives in AWS, so I started at the service (is it listening on IPv6?), and looked at local firewalls, then security groups, and worked my way out to the VPC. I checked NACLs and they were fine, as was all the stuff I checked along the way. It looked like everything should be working, but even a test as simple as connecting to the port via netcat from within the same VPC failed. I started looking at routing, and found the default route table for the VPC was misconfigured. It had a default route pointing to the internet gateway for IPv4, but no default route for IPv6 to hit the internet gateway. I added the route and did a quick check with `curl -6` to verify, before turning my attention back to my API client.

## TLS Is Also Broken?

Earlier on, I also discovered that the remote endpoint was misconfigured and sending back only the server certificate, not the chain including the intermediate CA. This worked fine in my browser and curl, but when I tried to connect via golang I had to pass a transport configuration to disable TLS checks. The astute reader would have seen this in my boilerplate `resty` code up top.

With `resty`:

{{< highlight go>}}
resty.SetTLSClientConfig(&tls.Config{InsecureSkipVerify: true})
{{< / highlight>}}

With `net/http`:

{{< highlight go>}}
tr := &http.Transport{TLSClientConfig: &tls.Config{InsecureSkipVerify: true}}
client := &http.Client{Transport: tr}
{{< / highlight>}}

This was a big red flag that something was wrong and prompted me to investigate further. I checked closer with our pal `openssl`:

{{< highlight bash>}}
openssl s_client -showcerts -connect coolapi.com:443
{{< / highlight >}}

I could see the server certificate, but not the issue/intermediate. I found the certificates for both the server and issuer, and created a proper chain:

{{< highlight bash>}}
cat server.pem issuer.pem > chain.pem ; chmod 600 chain.pem
{{< / highlight >}}

I installed this on the server and restarted the service, and I could take those terrible, insecure TLS configs out of play.

## Is Resty Faster?

After resolving the issue at the other end, I decided to try again with `resty` and with `net/http`. Both yielded similar results with a RTT of around a half a second for compiled binaries, while doing wither via `go run` was about a second. Much better.

{{< highlight bash>}}
csuttles@cs-mbp15:[~/src/golang-api-client]:
[Exit: 0] 09:01: go build -o resty restyghost.go && time ./resty 2>&1 > /dev/null

real  0m0.466s
user  0m0.072s
sys 0m0.019s
{{< / highlight >}}

## Summary

I learned some neat things digging into this issue and it was a relief to discover that my program was not failing to do something simple because of my own mistakes. It was also very fortunate that I had access to the remote endpoint to fix the problems I found, since often we do not have that access or luxury. I got valuable experience with the golang stdlib, and also got some quality time with my debugger. I improved both my skills and the remote endpoint. That's a win in my book.

