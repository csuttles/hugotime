+++
author = "Chris Suttles"
categories = ["containers", "jenkins", "AWS", "Docker", "CI"]
date = 2017-12-25T04:42:15Z
description = ""
draft = false
cover = "/images/2017/12/docker-jenkins.png"
slug = "using-jenkins-and-aws-to-build-and-push-docker-images"
tags = ["containers", "jenkins", "AWS", "Docker", "CI"]
title = "Using Jenkins and AWS to Build and Push Docker Images"

+++


This post will explore publishing a very simple Docker image to Docker Hub in a simple CI pipeline. I used a couple similar posts and documentation for reference while setting this up:

* [Building your first Docker image with Jenkins 2: Guide for developers](https://getintodevops.com/blog/building-your-first-docker-image-with-jenkins-2-guide-for-developers)
* [How To Build Docker Images Automatically With Jenkins Pipeline](https://blog.nimbleci.com/2016/08/31/how-to-build-docker-images-automatically-with-jenkins-pipeline/)
* [Using a Jenkinsfile ](https://jenkins.io/doc/book/pipeline/jenkinsfile/)

You can find my repository for this post here:

[https://github.com/csuttles/dockerci](https://github.com/csuttles/dockerci)

#### Overview

The CI pipeline built in this post is something like this:

* Our Dockerfile, application code and Jenkinsfile live in a git repository
* Jenkins polls this repository for changes, and runs the build if a new change has been pushed
* The build process starts on a local Jenkins master, which is also run in Docker, and grabs the Jenkinsfile when the repo is cloned locally
* The Jenkins master is configured to use AWS, and it spawns a Jenkins slave (if one is not already running) in in EC2 (a t1 micro); this instance is terminated after a specified timout.
* The AWS Jenkins slave clones the repository, builds the image and pushes to Docker Hub, tagging it with an incremental build number and also 'latest'

#### Configure Jenkins Master

I started out running my local Jenkins master by running the following command on one of my lab machines which is already running Docker:

{{< highlight markdown >}}

docker run --name jenkins -d -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts

{{< / highlight >}}

I grabbed the administrator password from the logs, logged in and ran through the installation wizard. You can see more details and screenshots of that part in the blog posts mentioned earlier. I installed the recommended plugins, the 'Green Balls' plugin, and also the one that I will focus on in this post: 

* Amazon EC2 Plugin

Before using this, you will need to configure credentials in for AWS in Jenkins, along with credentials for Docker Hub, which we will use later to push the image:

![Screen-Shot-2017-12-24-at-12.01.07-PM](/content/images/2017/12/Screen-Shot-2017-12-24-at-12.01.07-PM.png)

I used the `us-west-2` region for this, so I used the following AMI and initscript when configuring the Amazon EC2 plugin:

* AMI: `ami-bf4193c7`
* INIT:
{{< highlight markdown >}}

sudo yum update -y
sudo yum install -y docker git java-1.8.0-openjdk
sudo service docker start
sudo usermod -a -G docker ec2-user
sudo alternatives --set java /usr/lib/jvm/jre-1.8.0-openjdk.x86_64/bin/java

{{< / highlight >}}

The remainder of the configuration is pretty self explanatory and is also covered in more detail in the [How To Build Docker Images Automatically With Jenkins Pipeline](https://blog.nimbleci.com/2016/08/31/how-to-build-docker-images-automatically-with-jenkins-pipeline/) blog post. You start by adding a cloud of type "Amazon EC2", and configure credentials and other options. I mention the region, AMI and init script since I used Amazon Linux, and AMIs are region specific. I also found it useful to disable instance termination while troubleshooting, and I experienced errors with the plugin until I selected the `Connect by SSH Process` in advanced options. I also configured only a single executor to avoid spinning up too many instances in EC2.


#### Configuring the Build and Jenkinsfile

With AWS configured, I had to do the job configuration, and make some changes to the Jenkinsfile. The Jenkinsfile and app are heavily borrowed from [Building your first Docker image with Jenkins 2: Guide for developers](https://getintodevops.com/blog/building-your-first-docker-image-with-jenkins-2-guide-for-developers), with only slight changes to make the Jenkinsfile accomodate building in AWS. There's a lot of good details on the "Hello World" node app, the GUI parts of setting up the Jenkins job in that post as well, so I won't repeat it here. The Jenkins set up can also be automated; one way to do it can be found [here](https://blog.nimbleci.com/2016/10/11/how-to-deploy-jenkins-completely-pre-configured/). Instead of repeating stuff from my references, I'll focus on what is different.

At the time of pusblishing this post, the Jenkinsfile looks like this:

{{< highlight markdown >}}

node {
    def app

    stage('Clone repository locally') {
        /* Let's make sure we have the repository cloned to our workspace */

        checkout scm
    }
}

node('docker') {

    stage('Clone repository on slave') {
        /* Let's make sure we have the repository cloned to our workspace */

        checkout scm

    }

    stage('Build image') {
        /* This builds the actual image; synonymous to
         * docker build on the command line */

        app = docker.build("csuttles/hsl")
    }

    stage('Test image') {
        /* Ideally, we would run a test framework against our image.
         * For this example, we're using a Volkswagen-type approach ;-) */

        app.inside {
            sh 'echo "Tests passed"'
        }
    }

    stage('Push image') {
        /* Finally, we'll push the image with two tags:
         * First, the incremental build number from Jenkins
         * Second, the 'latest' tag.
         * Pushing multiple tags is cheap, as all the layers are reused. */
        docker.withRegistry('https://registry.hub.docker.com', 'docker-hub') {
            app.push("${env.BUILD_NUMBER}")
            app.push("latest")
        }
    }
}

{{< / highlight >}}
