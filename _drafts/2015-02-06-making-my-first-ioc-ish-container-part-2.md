---
title: Making My First IoC-ish container part 2
layout: post
author: MegaWubs
categories: 
---

This is the second post in a series of posts about the creation of my first IoC(ish) container. In this post I'll
go through each step of automatically initiating a class with it's dependencies.  

## Part 2, Initiation loop

At the hart of my IoC container is a initiation loop (a term I just came up with). It's basically a loop that keeps 
initiating classes until it's capable of initiating the requested class.