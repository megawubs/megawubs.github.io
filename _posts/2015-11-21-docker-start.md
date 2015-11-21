---
published: false
title: Docker Start
layout: post
author: MegaWubs
categories: PHP
---


## Docker Start

A lot hase changed since my last post! I've found a job, one that's really challeging and exactley the kind of job I was looking for. A tool we use is Docker. I emedeatly deep dived into it and this is my writeup of my understanding of Docker so far.

### What's Docker? You Ask
Docker is nothing more than a utility to enable a technique that's already possible for quit a while on a linux kernel. This technique is known as containers and a container is nothing more than one isolated process. A lot of  time containers are compared with the good old VM's (Virtual  Machines). This is in a sence true, but it's also completley different. Where a VM is a complete OS running on a hypervisor to mimic hardware, a container is running isolated on the same hardware as the guest OS. This all is possible since the release of Linux kernel 3.8. which introduced `namespaces` and `cgroups`. I wont go into how these two features make containers possible, just know that they do.

Since containers are a feature backed into your Linux kernel, where does Docker comes into play? Docker helps you build, run and deploy these containers.

### Build
I still remember the day's I had to configure my own server to have the right PHP version, install Mysql, setup apache and cross my fingers to see if I get a `it works!` page from apache. Almost always when I was done setting up my LAMP stack, I didn't dare to touch it again, afraid to break it. For personal projects this was somewhat ok, but whit a team developing on their local machines this becomes a pain really fast.

That's where a docker image comes into play. A docker image can be compared with an `.iso` of a Linux distro you download. It's a base image that can be used to create a container. A docker image is build with the instructions inside a `Dockerfile` The dockerfile declares the base image you want to derrife from, and has a few commands to install new software and configure the image. Once done, you can build it. This process will first download the base image you've defined, run all commands inside that image (here happens some container magic I wont' go into) and give the new created image a name. For example, the Dockerfile for your PHP image with xdebug installed could look like this:


{% highlight text %}
FROM php:5.6-apache

RUN pecl install -o -f xdebug  \
        && echo "zend_extension=/usr/local/lib/php/extensions/no-debug-non-zts-20131226/xdebug.so" > /usr/local/etc/php/conf.d/xdebug.ini
{% endhighlight text %}

The first line in a Dockerfile allways needs to be a `FROM` statement. This lets Docker know the image it needs do build upon. Next, we can choose a viarity of statements to execute, in this example the `RUN` statement executes the given command inside the container. This specific command installs xdebug and enables the extention. 

Other statements would be `VOLUME`, `COPY`, `USER` and `RUN`. `VOLUME` creates a volume that won't be deleted after a container is destoryed. `COPY` copy's files from the host to the image you're building, `USER` sets the user that runs the process and `RUN` tells Docker which process to run when the container is running.

You can build the image with the following command

{% highlight text %}
$ docker build -t my/php-dev .
{% endhighlight text %}

be sure to run this in the same folder as where the `Dockerfile` is located and don't forget the dot! This tells docker to look at the current directory for the Dockerfile.
Depending on your internet speed, Docker will start downloading the base image, in our case `php:5.6-apache`. When it's downloaded, you can see the output for the command to install xdebug on your screen. When it's done, Docker tells you something like: `Successfully built 43b9231335af` Where `43b9231335af` is the hash of the image.

### Run
Now that we have an image, we can run it! This is the awesome part, and one of the way's to ensure everyone has the same development environment.

To run the image you just build, simply execute the following command

{% highlight text %}
$ docker run my/php-dev
{% endhighlight text %}

And you'll see apache being loaded up. AWESOME!

Now you're probably wondering how to get this image to your co-workers or other intrested people. For this Docker created the Docker Hub. It's a place that you can copare with github, but for builded images not for code. You can create an account on hub.docker.com and push the image you've just created to your docker registery. I recomend to give your image another name than `my/php-dev` as the convention is to have `<vendor>/<useful-container-name>`. 

### Docker Hub
To see the power of the docker hub, execute the following command:

{% highlight text %}
$ docker run node
{% endhighlight text %}

This will pull in the latest node image from the docker hub and when it's done Docker will make a conatainer from it (aka, run it) and you'll see a node shell. This all is possible withoud installing anything from node on your machine!

### Deploy
The third part of what Docker can do for you, deploy, is something I don't have experience with yet. Hopefully I'll soon start invastigating this part of the Docker ecosystem.

## Docker And I
As I've said in the intro, I've recenty started using Docker. The process was quit intressting, as promised Docker should be all about being up and running quick, but for me this was not the case. I work on a Mac, and this has given me some headace. My solution to the problem was to run docker completley inside a Linux Mint VM, with also my code on the VM and localy. I let PHPStorm sync the changes to the VM. I did this because the filesytem mapping from Mac to the boot2docker machine is terrible slow. With the filesystem mapping, I had a page load of 1,5 sec. With my current setup the pageload is +/- 200 msec (depending on xdebug loaded or not).
