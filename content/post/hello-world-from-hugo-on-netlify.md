+++
title =  "Hugo on Netlify"
date = 2020-06-01T13:52:40-07:00
draft = true
author = "Chris Suttles"
categories = ["hugo", "netlify", "golang", "javascript"]
description = "Migrating from Ghost on AWS to Hugo on Netlify"
cover = "/images/2020/06/hugo-logo-wide.svg"
slug = "hugonetlify"
tags = ["hugo", "netlify", "golang", "javascript"]
aliases = ["/hugonetlify/"]
+++

Hello from [Hugo!](https://gohugo.io) on [Netlify](https://gohugo.io/hosting-and-deployment/hosting-on-netlify/)

I was running my blog on AWS for a long time, and using [Ghost](https://ghost.org/) to serve my content. That was nice for a while, but I had a lot of features I didn't care about and I felt as it grew it started to get too ["wordpressy"](https://wpscan.org/). I don't want a thousand widgets and WYSIWYG nonsense. I want an easy way to publish content, preferably via markdown. That's it.

After managing AWS stuff manually for my blog and thinking about the infrastructure and how much it will cost me, it was very refreshing to move to netlify and deploy things with git commits. I added a single config file to my repo and granted some permissions and we are off to the races.

{{< highlight toml >}}
[build]
publish = "public"
command = "hugo --gc --minify"

[context.production.environment]
HUGO_VERSION = "0.72.0"
HUGO_ENV = "production"
HUGO_ENABLEGITINFO = "true"

[context.split1]
command = "hugo --gc --minify --enableGitInfo"

[context.split1.environment]
HUGO_VERSION = "0.72.0"
HUGO_ENV = "production"

[context.deploy-preview]
command = "hugo --gc --minify --buildFuture -b $DEPLOY_PRIME_URL"

[context.deploy-preview.environment]
HUGO_VERSION = "0.72.0"

[context.branch-deploy]
command = "hugo --gc --minify -b $DEPLOY_PRIME_URL"

[context.branch-deploy.environment]
HUGO_VERSION = "0.72.0"

[context.next.environment]
HUGO_ENABLEGITINFO = "true"
{{< / highlight >}}

I won't go into detail on the deployment since it is very easy and explained well in the linked docs, but I thought it might be worthwhile to pause and discuss *why* I made these choices.

### Why Netlify

I chose Netlify over alternatives for deployment because the tooling is very simple and intuitive, and it has a lot of options that I can about as well as some I'm interested in for the future. There's no A/B testing your github pages, for example. There's also a lot of people using netlify and writing cool stuff there, so there's a nice ecosystem of useful build tools and plugins.

For example, in my netlify.toml, I've added the following build plugin with just two lines:

{{< highlight toml >}}
[[plugins]]
package = "netlify-plugin-is-website-vulnerable"
{{< / highlight >}}

That will break my build if I am using any any libraries in my code that have vulnerabilities associated with them. Imagine my surprise when it immidiately broken my build. Here's a snippet of the build log.

{{< highlight bash>}}
6:33:18 PM: ┌────────────────────────────────────────────────────────────────┐
6:33:18 PM: │ 3. onSuccess command from netlify-plugin-is-website-vulnerable │
6:33:18 PM: └────────────────────────────────────────────────────────────────┘
...
6:33:23 PM:   Libraries:
6:33:23 PM:     [*] jQuery 3.3.1
6:33:23 PM:     [*] AMP 2005150002001
6:33:23 PM:   Vulnerabilities:
6:33:23 PM:     ⎡ ✖ jQuery@3.3.1
6:33:23 PM:     ⎜ ■■■  3  vulnerabilities
6:33:23 PM:     ⎣ ▶︎ https://snyk.io/vuln/npm:jquery?lh=3.3.1
6:33:23 PM:   [3] Total vulnerabilities
6:33:23 PM:   [2325.06ms] execution time
6:33:23 PM:   vulnerabilities powered by Snyk.io (https://snyk.io/vuln?type=npm)
6:33:23 PM: ​
6:33:23 PM: ┌──────────────────────────────────────────────────────┐
6:33:23 PM: │ Plugin "netlify-plugin-is-website-vulnerable" failed │
6:33:23 PM: └──────────────────────────────────────────────────────┘
6:33:23 PM: ​
6:33:23 PM:   Error message
6:33:23 PM:   Error: site is vulnerable
6:33:23 PM: ​
6:33:23 PM:   Plugin details
6:33:23 PM:   Package:        netlify-plugin-is-website-vulnerable
6:33:23 PM:   Version:        1.0.8
6:33:23 PM:   Repository:     git+https://github.com/erezrokah/netlify-plugin-is-website-vulnerable.git
6:33:23 PM:   npm link:       https://www.npmjs.com/package/netlify-plugin-is-website-vulnerable
6:33:23 PM:   Report issues:  https://github.com/erezrokah/netlify-plugin-is-website-vulnerable/issues
6:33:23 PM: ​
6:33:23 PM:   Error location
6:33:23 PM:   In "onSuccess" event in "netlify-plugin-is-website-vulnerable" from netlify.toml
{{< / highlight >}}

*but I didn't even _use_ any libraries?!?!?!????*
{{< highlight bash>}}
(╯°□°)╯︵ ┻━┻
{{< / highlight>}}
I didn't use any libraries, but I *did* grab [an excellent theme](https://themes.gohugo.io/hugo-theme-dream/) and [fork it](https://github.com/csuttles/hugo-theme-dream) so that I could get some nice layout stuff without much work. It turns out that theme used an old version of jquery, which is what broke my build. A [quick update to the latest version](https://github.com/csuttles/hugo-theme-dream/commit/02368c84b7fe454afc8eff505e6ceb08150a4656) fixed it right up.

That's something I got with two lines of config; they really emphasize that netlify is user friendly and so far that has been my experience. This example is from just one of the [build plugins](https://docs.netlify.com/configure-builds/build-plugins/#install-a-plugin). There's also a/b testing and a lot of other features that just aren't available in alternatives like github pages, which was my 2nd choice.

## Why Hugo

I wanted to move away from managing any infra for my blog, and when I started looking at my options and reading feedback, it was as easy choice for me. I like [golang](https://golang.org/), so I thought it made a better choice for me than something like jekyll. There's lots of ways to generate a static site, so I wanted to choose something that was

* newer
* [open source](https://github.com/gohugoio/hugo)
* has a [strong](https://discourse.gohugo.io/) [community](https://github.com/gohugoio/hugo/pulse)
* personal appeal

 and hugo ticked all those boxes for me.

Newer is important to me because I want to invest my time wisely. Community is important to me because healthy, active OSS communities make great software and user experiences possible. The best software in the world is almost invariable attached to a large, diverse community of active developers, users and contributors. This makes change fast and positive and it also makes it easy to find help (or make contributions!).

One great example of the community around Hugo is the tool I used to import my content from Ghost. I used [ghostToHugo](https://github.com/jbarone/ghostToHugo) and a json export of my data to seed the new hugo repo for my blog. This gave me an a very good start, and I only had to do some minor refactoring to make things work with the theme I chose. This immediately reminded me of the benefit of moving to a static site generator instead of a DB driven CMS; I was able to do a few mass refactors with some simple scripting and tune things right up on local disk in my working directory (and fix them with the power of git if I messed up!). With a CMS, I might have had to resort to editing manually in some WYSIWYG (gross) or automating via curl/requests/selenium (less gross, but a lot more work than mass refactors on local disk). There's a big gap between something like this pseudo code example:

{{< highlight bash >}}
find . -type f -name '*.md' -exec perl -ne 'next if /^foo_bar = baz/; print' -i'' {} \;
{{< / highlight >}}

and something like this pseudo code example:

{{< highlight bash >}}
for post in $(find . -type f -name '*.md' -print | sed -e 's/.md$//')
do  
    TMPFILE=$(mktemp)
    curl https://somehost/api/getpost?slug=${post} | perl -ne 'next if /^foo_bar = baz/; print' | tee ${TMPFILE}
    curl -X POST https://somehost/api/editpost?slug=${post} --data @${TMPFILE}
    rm -f $TMPFILE
done
{{< / highlight >}}

The latter doesn't account for any kind of javascript or error handling and would probably either balloon into much more code, get scrapped and written in a scripting language, or produce inconsistent results.

## Summary

I'm excited about moving to a new publishing platform. I hope the new design is appealing and that the easy workflow will lead to more content. I also hope to improve my skills with javascript and golang along the way. If I get even part of what I hope, it will be a win for me. :)

Thanks for reading!
