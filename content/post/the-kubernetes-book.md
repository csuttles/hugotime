+++
author = "Chris Suttles"
categories = ["kubernetes", "books", "containers"]
date = 2017-11-13T10:23:05Z
description = ""
draft = false
cover = "/images/2017/11/Screen-Shot-2017-11-12-at-6.22.27-PM.png"
slug = "the-kubernetes-book"
tags = ["kubernetes", "books", "containers"]
title = "The Kubernetes Book"
aliases = ["/the-kubernetes-book/"]

+++


# Adequate

I bought and recently read [The Kubernetes Book](https://www.amazon.com/Kubernetes-Book-Nigel-Poulton/dp/1521823634/), and it was not great, but also not terrible; it was just OK.

## Pros

It's pretty cheap (~$12.00 on Amazon at the moment). It's a decent whirlwind tour of Kubernetes. Nothing goes very deep, but the concepts presented are accurate. The book introduces different deployment types (minikube, single node, and your basic HA/PROD design), as well as building blocks of Kubernetes. The introduction of the API, Pods, Replication Controllers, Replica Sets, and Deployments are all easy to follow and describe them in sufficient detail through simple examples. There is not enough information in this book to run Kubernetes in production, but the examples work well enough for understanding the concepts. You deploy stuff in minikube, you walk through templates and apply them, and do basic deployment stuff (like a rollback). It's almost exactly the exercises in [the Kubernetes basics tutorial](https://kubernetes.io/docs/tutorials/kubernetes-basics/). There's some coverage of labels, and there's enough here to understand Kubernetes well enough for local development, and enough to give you a solid base when you buy another book, course, or do soem reading and practice.

## Cons

This book was riddled with spelling errors. There were a few places where an entire word was wrong. It seemed as though the book was edited in a hurry.

There's also just not a lot of meat. The book is more like a magazine, at only about 1/4 of an inch thick. There's the boring "why is it called Kubernetes?" and "why do people write k8s instead of kubernetes?" stuff to start with, which is a real waste of time and space since that information is easily found elsewhere and is not essential to actually *using* Kubernetes. That's pretty absurd when you consider the size of this book.

There's no coverage of config maps (or if there is it was short enough for me to forget), which is a pretty big missing piece.

There's no coverage of annotations.

## Summary

If you are totally new to Kubernetes, and you *really* want to buy a book, this one is OK. There are better choices. If you have even a cursory understanding of Kubernetes, I'd skip this and buy a different book. You can also easily get everything in this book by simply [reading the documentation](https://kubernetes.io/docs/home/). I also recommend checking out [Kubernetes the hard way](https://github.com/kelseyhightower/kubernetes-the-hard-way). It was handy for me on a long flight with no WiFi, so it served it's purpose and is now being resold on Amazon.

