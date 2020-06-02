+++
title =  "Hugo on Netlify"
date = 2020-06-01T13:52:40-07:00
draft = true
author = "Chris Suttles"
categories = [""]
description = ""
cover = "/images/2020/06/hugo-logo-wide.svg"
slug = "hugonetlify"
tags = [""]
aliases = ["/hugonetlify/"]
+++

Hello from [Hugo!](https://gohugo.io) on [Netlify](https://gohugo.io/hosting-and-deployment/hosting-on-netlify/)

I was running my blog on AWS for a long time, and using [ghost](https://ghost.org/) to serve my content. That was nice for a while, but I had a lot of features I didn't care about and I felt as it grew it started to get too "wordpressy". I don't want a thousand widgets and WYSIWYG nonsense. I want an easy way to publish content I author. That's it.

After managing AWS stuff manually for my blog and thinking about the infra and how much it will cost me, it was very refreshing to move to netlify and deploy things with git commits. I added a single config file to my repo and granted some permissions and we are off to the races.

I won't go into detail on the deployment since it is very easy and explained well in the linked docs, but I thought it might be worthwhile to pause and discuss *why* I made these choices, instead of just declaring what I have done. "Look upon my mighty works!"

<!--
<img src="https://media.giphy.com/media/3oKIPcfX631trLEyCQ/source.gif" alt="I am a golden god" />
-->

