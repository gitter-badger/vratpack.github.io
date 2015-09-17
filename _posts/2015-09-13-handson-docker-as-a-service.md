---
layout: post
title: "HandsOn: Docker and Coopto - Docker-as-a-Service in vRealize Automation"
date: 2015-09-13 08:00:00
comments: true
tags: [author-chk, vra, vro, vmware, docker]
author:
    name: "Christian Klose"
    twitter: "christianklose"
excerpt_separator: <!--more-->
---



<!--more-->

# Short Intro

[Robert Szymczak](https://vratpack.com/about/) created a Plug-In for VMware vRealize Orchestrator which should help Administrators and Users to easily deploy and manage Docker Instances, Containers and Nodes on VMware Infrastructure. The project is open source and is located on [GitHub](https://github.com/m451/coopto)  and there is also a [ReadMe](https://github.com/m451/coopto/blob/master/README.md) and a [Wiki](https://github.com/m451/coopto/wiki)  available. You could also download it as a vRealize Orchestrator Package from VMware Solution Exchange [here](https://solutionexchange.vmware.com/store/products/coopto-a-docker-plug-in-for-vmware-vrealize-orchestrator--2).

# Install to Demo Environment

I have successfully installed the PlugIn and the Package File and i have a Docker Host on CoreOS running Server Version 1.4.1 with API Version 1.16.

But you can use any other OS or Docker distribution. Install instructions are available at [docker.com](https://docs.docker.com/installation/)

Check your version of Docker by typing ***docker version***

![Image of Docker Version](/assets/posts/2015-09-13-handson-docker/05-01-_2015_16-12-19.png)

# Overview

In vRO Workflow Library you see the new menu Coopto which is structured into four sections

- Configuration
- Containers
- Images
- Sandbox

![Coopto - Docker Plugin for vRealize Orchestrator](/assets/posts/2015-09-13-handson-docker/05-01-_2015_16-22-12.png)

In *Configuration* you can Add and Remove Docker Nodes / Hosts. In *Containers* you can manage your Containers on the specified Node and on *Images* you can handle images from Dockerhub. *Sandbox* is a section to show you how you can predefine Applications using Docker Containers.

There are also some Actions coming with the Coopto Plug-In and the Coopto Package which are needed for the predefined Workflows and Sandbox Samples.

![Coopto - Docker Plugin for vRealize Orchestrator](/assets/posts/2015-09-13-handson-docker/05-01-_2015_16-54-42.png)

Let’s get started with adding a new Docker Node by simply starting ***Add Docker Node*** from the Configuration Section

![Coopto - Docker Plugin for vRealize Orchestrator](/assets/posts/2015-09-13-handson-docker/05-01-_2015_16-33-51.png)

After successful Workflow run check the vRO Inventory Tab and expand the Coopto Menu and select the Node you added before. The Status in the General Tab should be ONLINE!

![Coopto - Docker Plugin for vRealize Orchestrator](/assets/posts/2015-09-13-handson-docker/05-01-_2015_16-37-22.png)

As you can see in the screenshot the Node was added and it scans for existing Docker Images on that Host which are shown under the Host in the Inventory.

By simply running ***docker images*** on the Docker Host you can double-check the downloaded Images.

![Coopto - Docker Plugin for vRealize Orchestrator](/assets/posts/2015-09-13-handson-docker/05-01-_2015_16-46-18.png)

# Use case / Example
As an quick and easy test we use one of the prepared Workflows Create Web SSH Console in the Sandbox Section coming with the Plug-In.

Let’s asume we did not have access to a SSH Client but we wanted to connect to a Linux Host. Therefore we can use wetty, a web based SSH console which is based on ChromeOS’ terminal emulator (hterm). You’re wright – this only works with a Chrome Browser :-)

![Coopto - Docker Plugin for vRealize Orchestrator](/assets/posts/2015-09-13-handson-docker/05-01-_2015_17-09-37.png)


After successfull workflow run switch to the Logs tab and copy the URL highlighted in the screenshot and paste it to a Chrome Webbrowser

![Coopto - Docker Plugin for vRealize Orchestrator](/assets/posts/2015-09-13-handson-docker/05-01-_2015_17-12-26.png)


et voilà – we have a connection to the Linux Host.

![Coopto - Docker Plugin for vRealize Orchestrator](/assets/posts/2015-09-13-handson-docker/05-01-_2015_17-16-25.png)

# RealWorld Integration to vRealize Automation

So now we come to the most interesting part of the story - bring technology to the people so everyone can use it even when he doesn't have access or knowledge of docker.

The ShowCase is to enable User to request a docker application through vRealize Automation Portal.

## vRealize Automation - Advanced Services

If everything is fine and running in vRealize Orchestrator let's start with the **Advanced Services** in vRealize Automation.

-----------

### Custom Resources
To manage Docker Containers through vRealize Automation we have to create a **Advanced Services > Custom Resource** with Orchestrator Type *Coopto:DockerContainer* which could be selected by only typing *coo*. Give it a Name like *DockerContainer* and some Description. For this demo i leave the Details From as it is but for a real scenario i would hide all fields which are not necessary to the user.

![Coopto - Docker Plugin for vRealize Orchestrator](/assets/posts/2015-09-13-handson-docker/SNAG-0000.png)

![Coopto - Docker Plugin for vRealize Orchestrator](/assets/posts/2015-09-13-handson-docker/SNAG-0001.png)

![Coopto - Docker Plugin for vRealize Orchestrator](/assets/posts/2015-09-13-handson-docker/SNAG-0002.png)

### Service Blueprints
As a first Service Blueprint i wanted to add on of the prepared Coopto Workflows from the Sandbox folder in the Coopto Plug-in.
Start with **Advanced Services > Service Blueprints** and click **Add** to create a new Service.

![Coopto - Docker Plugin for vRealize Orchestrator](/assets/posts/2015-09-13-handson-docker/SNAG-0004.png)

Browse the vRealize Orchestrator Library to **Library : Coopto : Sandbox** and Select the *Create Web SSH Console* Workflow.

![Coopto - Docker Plugin for vRealize Orchestrator](/assets/posts/2015-09-13-handson-docker/SNAG-0005.png)

Give a Name, Description and maybe Version to the Blueprint.
I love the possibility to hide this ugly catalog request information page from the Service Request.

![Coopto - Docker Plugin for vRealize Orchestrator](/assets/posts/2015-09-13-handson-docker/SNAG-0006.png)

As in the Custom Resource Form i keep all information in this Blueprint form which come from the Workflow.

![Coopto - Docker Plugin for vRealize Orchestrator](/assets/posts/2015-09-13-handson-docker/SNAG-0008.png)

An important step is to bind the Output Parameter from the Workflow to a Provisioned Resource. Otherwise you did not receive any manageable Items in vRealize Automation.


### Catalog Management
For Coopto i created a *Services Group* in **Administration > Catalog Management > Services** and i linked the created Service Blueprint *Create Web SSH Console* to this Services Group. A nice new logo and then we can preview our new Service in the **Catalog** View.

![Coopto - Docker Plugin for vRealize Orchestrator](/assets/posts/2015-09-13-handson-docker/SNAG-0013.png)

![Coopto - Docker Plugin for vRealize Orchestrator](/assets/posts/2015-09-13-handson-docker/SNAG-0014.png)

### New Request
Let's request one container through vRealize Orchestrator and  we wanted to use this Container as a SSH Client to connect to a SSH Host with the *IP 10.111.6.212 Port 22* and *User root*.

You can give a Name to the Container which later also appears in the **Items list**.

![Coopto - Docker Plugin for vRealize Orchestrator](/assets/posts/2015-09-13-handson-docker/SNAG-0015.png)

### Items
After a few seconds check the **Items List** for the Name given to the Container in the previous request.
Click on the Name to view the Details of this Item.

![Coopto - Docker Plugin for vRealize Orchestrator](/assets/posts/2015-09-13-handson-docker/SNAG-0016.png)

![Coopto - Docker Plugin for vRealize Orchestrator](/assets/posts/2015-09-13-handson-docker/SNAG-0017.png)

### New Resource Action : Remove Container
Now that we have created a Docker Container as Custom Resource managed by vRealize Automation we should create some Resource Actions specially for the management of Docker Container Items.

Go to **Advanced Services > Resource Actions** and Click *Add* to create a new Action.


![Coopto - Docker Plugin for vRealize Orchestrator](/assets/posts/2015-09-13-handson-docker/SNAG-0009.png)

Browse the vRealize Orchestrator Library to **Library : Coopto : Containers** and Select the *Remove Container* Workflow.

![Coopto - Docker Plugin for vRealize Orchestrator](/assets/posts/2015-09-13-handson-docker/SNAG-0010.png)

Define the Resource type and the Input parameter to match with the Custom Resource created earlier.

![Coopto - Docker Plugin for vRealize Orchestrator](/assets/posts/2015-09-13-handson-docker/SNAG-0011.png)
Keep the Name and Description and give a Version to the action. Don't forget to check *Hide catalog request information page* - if you want.
You can also check the Type : *Disposal* and as the Target criteria select *Always available*.

![Coopto - Docker Plugin for vRealize Orchestrator](/assets/posts/2015-09-13-handson-docker/SNAG-0012.png)

Keep the form Elements and click **Add**.

### Resource Action : Get Connection in vRA

For some Applications in Docker Container it is helpfull to get the URL or Connection string.  Our example Service Container Web SSH Console to connect to a SSH Host creates such a connection string but unfortunately it is not redirected to the vRealize Automation Portal directly.

A Resource Action could be the solution ...

![Coopto - Docker Plugin for vRealize Orchestrator](/assets/posts/2015-09-13-handson-docker/SNAG-0018.png)

Browse the vRealize Orchestrator Library to **Library : Coopto : Containers : vRA ** and Select the *Get Connection in vRA* Workflow.

![Coopto - Docker Plugin for vRealize Orchestrator](/assets/posts/2015-09-13-handson-docker/SNAG-0019.png)

Map the Resource Action to the Custom Resource of the Docker Container.

![Coopto - Docker Plugin for vRealize Orchestrator](/assets/posts/2015-09-13-handson-docker/SNAG-0020.png)

Give a Name, Description and Version...

![Coopto - Docker Plugin for vRealize Orchestrator](/assets/posts/2015-09-13-handson-docker/SNAG-0021.png)

You can define Prefix and Postfix if you want and sometimes it make sense to remove or hide these fields.
In my example i normally would define *http://* as a Prefix but this is currently not in the Screenshots.

![Coopto - Docker Plugin for vRealize Orchestrator](/assets/posts/2015-09-13-handson-docker/SNAG-0022.png)

You have to publish and configure the Actions to the Entitlement before they are visible to the Items Actions menue!

Select your Container for the *Web SSH Console* and Click **Actions > Get Connection in vRA**

![Coopto - Docker Plugin for vRealize Orchestrator](/assets/posts/2015-09-13-handson-docker/SNAG-0023.png)

You don't have to define a Prefix or a Postfix as you don't need them in this example. Click **Next**.

![Coopto - Docker Plugin for vRealize Orchestrator](/assets/posts/2015-09-13-handson-docker/SNAG-0024.png)

In this Output - Form you can see the link to my Docker Host and a Port which is mapped into the Container i created before. If i had configured a Prefix like http:// i could directly click on the URL to open the connection.
I have to copy the URL s00-lxc-01:49155 and paste it to a Chrome Browser...

![Coopto - Docker Plugin for vRealize Orchestrator](/assets/posts/2015-09-13-handson-docker/SNAG-0025.png)

and again : it works
I'm now connected to the Container on my Docker Host which has created a SSH Connection to the ip i configure in my Request.

![Coopto - Docker Plugin for vRealize Orchestrator](/assets/posts/2015-09-13-handson-docker/SNAG-0026.png)

Last step is to go back to my Items View, select my Container and remove it with the created Resource action.

Further information about Docker, Photon and VMware Cloud-Native Apps could be found [here at VMware](http://www.vmware.com/cloudnative/cloudnative) or as an [Blogpost by Robert Szymczak](/2015/09/02/Docker-on-VMware.html).


*Have some fun with Coopto, vRealize Orchestrator and vRealize Automation.*
