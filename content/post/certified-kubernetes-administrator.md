+++
author = "Chris Suttles"
categories = ["kubernetes", "certification", "CNCF", "containers", "Docker"]
date = 2018-12-10T00:55:49Z
description = ""
draft = false
cover = "/images/2018/12/logo_cka.png"
slug = "certified-kubernetes-administrator"
tags = ["kubernetes", "certification", "CNCF", "containers", "Docker"]
title = "Certified Kubernetes Administrator"

+++


I just passed the CNCF (Cloud Native Computing Foundation) [CKA](https://www.cncf.io/certification/cka/) (Certified Kubernetes Administrator) exam. Here's some information about how I prepared for the exam and a few tips for people interested in taking the exam.

## Starting From Scratch

For folks who are totally unfamiliar with k8s, edX offers a [free, self paced course](https://www.edx.org/course/introduction-to-kubernetes), which is a great place to start. There's also a list of online training available [in the official k8s docs](https://kubernetes.io/docs/tutorials/online-training/overview/).

## Minikube

The most important kind of preparation for any exam, especially a _practical_ exam, like the CKA is getting your hands dirty. [Minikube](https://github.com/kubernetes/minikube) is a great way to use k8s locally. It's trivial to install and use, and can help you get comfortable with the api-resources you will need to know for the exam.

## CNCF Training

The CNCF offers a self paced course, and it is excellent. This was the best resource for learning k8s. I feel I could have passed the exam with just this course and hands on time if I had focused on preparing for long enough.

## Kubernetes the Hard Way

This is basically a right of passage from "I'm interested in k8s" to "I know some things about k8s". Once you have an understanding of building clusters with `kubeadm`, `kops` or similar tools, it's definitely worth going through [kubernetes the hard way](https://github.com/kelseyhightower/kubernetes-the-hard-way). It's imperative that you do not simply copy and paste the examples, but try to understand them. If all you do is copy and paste here, you won't learn anything, and that's the whole point. Thinking about what you are doing as you build CSRs and request certificates from the k8s CA, and then where they actually get installed will help solidify your understanding of the communication flow within the k8s [architecture](https://kubernetes.io/docs/concepts/architecture/).

## Get Your Hands Dirty

Because the [CKA](https://www.cncf.io/certification/cka/) is a practical exam, I feel this is the best kind of preparation. You need to be able to manage workloads ([Pods](https://kubernetes.io/docs/concepts/workloads/pods/) and [Controllers](https://kubernetes.io/docs/concepts/workloads/controllers/)) and [Services](https://kubernetes.io/docs/concepts/services-networking/service/). It's important to get familiar with the [documentation](https://kubernetes.io/docs/home/?path=users&persona=cluster-operator&level=foundational), since you _can_ reference the official docs (in a single tab) during the exam. Get to know the `kubectl` command intimately; the `kubectl explain` subcommand is something worth exploring deeply since you can use it to get the definitions for all api-resource types from the command line. This can really help when you are trying to write a spec and get stuck. For a quick example, let's look at the output of `kubectl explain pod.spec.affinity`:

{{< highlight markdown >}}
KIND:     Pod
VERSION:  v1

RESOURCE: affinity <Object>

DESCRIPTION:
     If specified, the pod's scheduling constraints

     Affinity is a group of affinity scheduling rules.

FIELDS:
   nodeAffinity	<Object>
     Describes node affinity scheduling rules for the pod.

   podAffinity	<Object>
     Describes pod affinity scheduling rules (e.g. co-locate this pod in the
     same node, zone, etc. as some other pod(s)).

   podAntiAffinity	<Object>
     Describes pod anti-affinity scheduling rules (e.g. avoid putting this pod
     in the same node, zone, etc. as some other pod(s)).
{{< / highlight >}}

You can look at any api resource this way, and drill down as far as you like, using the dot delimited notation.

Here's some other useful things you can do with `kubectl` include:

* `kubectl run --image=nginx nginx --replicas 2 --dry-run -o yaml > nginx.yaml` This will give you a deployment yaml template that you can edit as needed before the old `kubectl create -f <file>`
* `kubectl get svc nginx-service --export -o yaml > nginx-svc.yaml` This will write the details of an _existing_ service to a yaml file. The `--export` flag will strip all the cluster specific info, so you can edit this file and `kubectl apply -f <file>` to update that service or use it as a template for creating new services.

Hands on experience like this will help you beat the clock and save time for questions that take longer.

## What to Expect

It's a practical exam, so you should be prepared to work on k8s clusters. You will have 3 hours to complete the exam, and it's proctored (very strictly) online. There's a terrific amount of detail about the exam in both the [candidate handbook](https://www.cncf.io/certification/candidate-handbook), and the [exam tips](https://www.cncf.io/certification/tips) available from the CNCF. While almost every certification encourages you to read the FAQ and associated documents, I have to emphasize how important and useful it is for this exam. Do yourself a favor, and read those documents carefully.There's also the [CNCF curriculum repo on github](https://github.com/cncf/curriculum), which is incredibly useful for understanding the breakdown of scoring. This helped me a lot in deciding where I needed to focus.

Every question provides a `kubectl config use-context <context>` command in the beginning, so you can make sure you operate on the right cluster. I made sure to run this on every question, even though they appear in order to minimize context switching. It's easy to run it and doing so means you never have to second guess.

