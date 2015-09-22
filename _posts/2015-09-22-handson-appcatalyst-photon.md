---
layout: post
title: "HandsOn: Appcatalyst and Photon"
date: 2015-09-22 23:00:00
comments: true
tags: [author-chk, vmware, docker, photon, apple, macosx]
author:
    name: "Christian Klose"
    twitter: "christianklose"
excerpt_separator: <!--more-->
---

If you are using Apple Mac OS and wanted to test-drive docker container there is a simple solution for that coming from VMware called AppCatalyst. I'm using VMware Fusion but i wanted a smarter solution to deploy Docker images like e.g. *boot2docker* does. So AppCatalyst gives you the power of the VMware hypervisor technology with an easy to use CLI and API to manage your VMs. As a little Background or Use-Case i wanted to run the docker image *greencape/jekyll* to test drive Markdown and Jekyll for my local writing of this blog. So we have a Use-Case and we have a technical solution for that - let's start with the download...

<!--more-->


At first you have to download [VMware AppCatalyst](https://www.vmware.com/cloudnative/appcatalyst-download) and install the PKG-File to your Mac.

You should set the AppCatalyst BIN Directory */opt/vmware/appcatalyst/bin* as PATH Variable to */etc/paths*.

<figure>
  <img src="/assets/posts/2015-09-22-handson-appcatalyst-photon/appcatalyst00.png" >
  <figcaption>Fig 1. Quick Overview of Appcatalyst and first test of Docker Image vmwarecna/nginx:latest</figcaption>
</figure>

As you can see on this Terminal Screenshot - there are only a few Operations available for the appcatalyst command. And you need only a few seconds to get a Docker image up and running.

<figure>
  <img src="/assets/posts/2015-09-22-handson-appcatalyst-photon/appcatalyst01.png" >
  <figcaption>Fig 2. First test of Docker Image vmwarecna/nginx:latest</figcaption>
</figure>

<figure>
  <img src="/assets/posts/2015-09-22-handson-appcatalyst-photon/appcatalyst02.png" >
  <figcaption>Fig 3. List of command operations available</figcaption>
</figure>

Usage : **appcatalyst [domain] [command]**

Virtual Machine Operations:

- **appcatalyst vm list**
- **appcatalyst vm create [id]**
- **appcatalyst vm clone [parentId] [id]**

Virtual Machine Power Operations:

- **appcatalyst vmpower list**
- **appcatalyst vmpower on [id]**
- **appcatalyst vmpower off [id]**
- **appcatalyst vmpower shutdown [id]**
- **appcatalyst vmpower suspend [id]**
- **appcatalyst vmpower pause [id]**
- **appcatalyst vmpower unpause [id]**

Guest OS Operations:

- **appcatalyst guest getip [id]**

<figure>
  <img src="/assets/posts/2015-09-22-handson-appcatalyst-photon/appcatalyst03.png" >
  <figcaption>Fig 4. Create and start the first Docker Host</figcaption>
</figure>

Once you have AppCatalyst up and running let's create the first Docker Host based on VMware Photon TP2 coming with the August Release of AppCatalyst.

<figure>
  <img src="/assets/posts/2015-09-22-handson-appcatalyst-photon/appcatalyst04.png" >
  <figcaption>Fig 5. Get IP of Guest OS</figcaption>
</figure>


<figure>
  <img src="/assets/posts/2015-09-22-handson-appcatalyst-photon/appcatalyst05.png" >
  <figcaption>Fig 6. SSH to the Guest OS</figcaption>
</figure>

<figure>
  <img src="/assets/posts/2015-09-22-handson-appcatalyst-photon/appcatalyst06.png" >
  <figcaption>Fig 7. Docker Host and API is ready - start Docker commands at your choice...</figcaption>
</figure>

*Have some fun with AppCatalyst, Photon and Docker!*

More Information:
[Project Bonneville - Application Containers with VMware vSphere](https://www.youtube.com/watch?v=q0Xg7mVOO8o)


What else to do:

Have to manage the Photon Host and it's IP address to avoid regular IP changes. I found this blog post by Florian Grehl at [virten.net](http://www.virten.net/2015/04/basic-commands-for-vmware-photon-and-docker/) about changing the IP address on a Photon Host.
