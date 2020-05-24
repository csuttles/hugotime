+++
author = "Chris Suttles"
categories = ["serverless", "faas"]
date = 2017-12-14T14:51:49Z
description = ""
draft = false
cover = "/images/2017/12/Screen-Shot-2017-12-13-at-10.06.03-PM.png"
slug = "fnproject-and-open-source-faas"
tags = ["serverless", "faas"]
title = "Fn Project and Open Source FaaS"

+++


I decided to take a [attend a Meetup on Serverless (Oracle Fn project)](https://www.meetup.com/Microservices-and-Cloud-Native-Architectures-SF-Bay-Area/events/245450847/), and it was a lot of fun. The first talk was excellent, and that will be the focus of this post.

#### Fn Presentation

The talk started with some history on the speaker, [Chad Arimura](https://twitter.com/chadarimura?lang=en), and some of the companies he's launched, as well as a few obligatory slides to level set things ("What is serverless?"). After the brief pre-amble, Chad got into the meat of the presentation, which was close to the quickstart on the [Fn github page](https://github.com/fnproject/fn). There were some important additions, like "hot functions", and the presentation included gracefully fielding some questions like "Why is this a thing?".

Chad pointed people to this post on Medium several times for understanding the purpose and application of Fn: 

> [8 Reasons why we built the Fn Project](https://medium.com/fnproject/8-reasons-why-we-built-the-fn-project-bcfe45c5ae63)

The "put it on a bumper sticker" explanation was "multi-cloud, open source lambda", and that was good enough to get my attention.

Incidentally, "hot functions" are functions that persist once the container they are run in is built. This avoids the penalty of container build on invocation, so if you are say, serving an endpoint from Fn, you want to use a hot function so that the first invocation is slow (700ms in the demo), but subsequent calls to your endpoint are much more reasonable response times (50-70ms in the demo).

There's a lot more, like a quick dive into the architecture. The [docs on running Fn in production](https://github.com/fnproject/fn/blob/master/docs/operating/production.md) have a good overview that looks a lot like the architecture slides from the talk. There's integration with [Prometheus & Grafana](https://medium.com/fnproject/announcing-prometheus-metrics-from-fn-2d0f9ddf0f09) as well as other monitoring solutions, and [Helm charts for Kubernetes deployment](https://medium.com/fnproject/fn-project-helm-chart-for-kubernetes-e97ded6f4f0c). There was a also a brief foray into [Flow](https://medium.com/fnproject/flow-101-be7f328ffce2), and a cool demo where a Flow glued together a lot of different functions asynchronously that were dependent on each other; the end result was a flow that scraped Flickr for pictures of cars, then kicked off jobs to detect the license plates in the images and render rectangles highlighting them, as well as run them through OCR to detect the plate, and finally, post to the updated images to Slack along with the plate. The only thing missing to make this a super valuable tool to your local law enforcement was a step to query the plate and see if it is stolen. Alternatively, the flow could also post the end result content to Twitter.

#### Post Meetup Follow Up

The talk sparked my interest. I decided to run through the quickstart and a few things in the github repo to  check it out. The quickstart is crazy easy.

I had most of the prerequisites, so I did:

```
brew install fn
fn start
mkdir -p fnproject/hello
cd $!
```

Then I created a simple golang program based on the docs with this content, named `func.go`:

```
package main

import (
  "fmt"
)

func main() {
  fmt.Println("Hello from Fn!")
}
```

Next I ran:

```
fn init
export FN_REGISTRY=<my docker registry>
fn run
fn deploy --app myapp
```

Then I was calling my function (func.go) with:

```
curl http://localhost:8080/r/myapp/hello
# or:
fn call myapp /hello
```

There's also a built in dashboard, which was easily launched with:

```
docker run --rm -it --link fnserver:api -p 4000:4000 -e "FN_API_URL=http://api:8080" fnproject/ui
```

It was easy to see invocations, and more detail in the terminal too:

![Screen-Shot-2017-12-13-at-10.03.14-PM-1](/content/images/2017/12/Screen-Shot-2017-12-13-at-10.03.14-PM-1.png)

The right pane of tmux is the result of `fn calls list myapp`.

#### Summary

While there are other contenders that are similar, like [Kubeless](https://github.com/kubeless/kubeless) or [OpenFaaS](https://github.com/openfaas/faas), I thought the Fn Project was impressive. Admittedly, I haven't spent a lot of time digging into the alternatives, and mention them mostly for completeness sake. If you are in a committed relationship with Kubernetes, Kubeless might be a better choice due to its tight integration with Kubernetes; it uses CRDs, which is also on the roadmap for Fn, but it's in Kubeless right now. OpenFaaS looks like it has a lot of similar features (including Flow), but I think the documentation for Fn Project seems a bit more clear and complete, and it looks a bit more modular in design, with multiple options for underpinning services like database and message queue. Overall, if you are considering deploying (or contributing to!) a FaaS framework, the Fn Project is worth a look.

