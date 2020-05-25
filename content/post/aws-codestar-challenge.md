+++
author = "Chris Suttles"
categories = ["AWS", "automation", "Python"]
date = 2017-09-13T11:02:50Z
description = ""
draft = false
cover = "/images/2017/09/codestar-example-dashboard.png"
slug = "aws-codestar-challenge"
tags = ["AWS", "automation", "Python"]
title = "AWS CodeStar Challenge"
aliases = ["/aws-codestar-challenge/"]

+++


I've exceeded the AWS free tier, and happened upon a post from a friend that leads to an easy $50 AWS credit. Introducing the [AWS Codestar Challenge](https://aws.amazon.com/codestar/codestar-credit-challenge/).

Codestar follows the pricing model of CloudForms and Elastic Beanstalk. It's free, but the resources created by CodeStar follow their respective pricing models. CodeStar brings together a bunch of AWS services to form an incredibly easy to use CI/CD pipeline. I was able to deploy a CI/CD pipeline for a Lambda based Python app in minutes, [including setting up my access](https://docs.aws.amazon.com/codecommit/latest/userguide/setting-up-without-cli.html?icmpid=docs_acs_console_connect#setting-up-without-cli-add-key).

The only thing I can see as a potential drawback is some of the shortcomings of CodeCommit that are mentioned in [this video](https://www.youtube.com/watch?v=rrbp-IVwFGY). If you want to skip the video, what I'm referring to is the lack of native code review capability in CodeCommit. You can, however [integrate Review Board](https://aws.amazon.com/blogs/devops/integrating-aws-codecommit-with-review-board/), as you might with a standalone git repo. While you get code review 'for free' with github, at many enterprises, integration with [Review Board](https://www.reviewboard.org/) or [Phabricator](https://github.com/phacility/phabricator) is the standard practice, so it's really not that big of a hurdle in my opinion.

More importantly, I set up a CI/CD pipeline in minutes, without managing Hudson or Jenkins, and the choices for for types of software projects you can deploy is plentiful. If you are deploying code into AWS, particularly for testing, CodeStar is a great way to get started developing and deploying your project via CI/CD very easily.

*Update 10/7/2017: Got my credit about three weeks ago. Thanks AWS!*

