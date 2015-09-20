---
layout: post
title: "Docker-As-A-Service - Running containers in VMware"
date: 2015-09-20 21:00:00
comments: true
tags: [author-rszy, vra, vro, docker, vmware, coopto]
author:
    name: "Robert Szymczak"
    twitter: "rszzy"
excerpt_separator: <!--more-->
description: "Running containers in VMware"
---
This post essentially is the translation of [my article](http://www.it-administrator.de/themen/virtualisierung/fachartikel/191523.html) for the German IT magazine *IT Administrator* as first mentioned [here](https://vratpack.com/2015/09/02/Docker-on-VMware.html). If you find this interesting you'll also like the posts by my colleague [Christian Klose](https://vratpack.com/blog/tag/author-chk/) and his experience with the Coopto plug-in.

<!--more-->

#Docker-As-A-Service - Running containers in VMware

Software eats the world. This statement from Marc Andreessen comes true particularly if you got a development department within your company. Admins see it: more and more open source technology like Docker is pushed into IT operations by the users.

Within IT operations a virtual infrastructure, preferably built on VMware, was the holy grail for a long time. And till today VMware is popular when it comes to enterprise IT.
Beyond doubt: many things that virtual machines (VMs) do today can be done with containers more efficiently and they even can adopt better to certain use-cases. Especially when looking at use-cases like virtual desktops (VDI) it becomes clear however that containers can't replace virtual machines entirely just yet.

But running both technologies in parallel is expensive and Docker, compared to virtualization, still isn't a mature technology. So what's the right strategy? Providers like Amazon combine both technologies and run Docker-Containers inside of VM instances. The advantage is clear: features and infrastructure of the already in-place virtual environment can be used without any change.

In the following we'll show you how to run container services within your companies existing VMware infrastructure and combine both technologies in a meaningful way.

<figure>
  ![Only one example of combining both technologies: scale-out and maintenance without downtimes](/assets/posts/2015-09-20-docker-as-a-service/scale.png)
  <figcaption>Fig 1. Only one example of combining both technologies: scale-out and maintenance without downtimes</figcaption>
</figure>

In theory you can use any of the many Docker Linux distributions as a VM to run within your vSphere. It's still worth, in terms of interoperability and future integration, to check what VMware itself has to offer. In the beginning of this year VMware presented a minimal open source Linux distribution called Photon which is nothing more then a container runtime. Besides VMware Tools, which are already installed in the distribution, Photon comes with many interfaces for upcoming container integrations from VMware (e.g. Lightwave). You can download this robust alternative to the many Docker distributions [from Bintray](https://bintray.com/vmware/photon/iso/view#files).

##First steps - taking Photon for a walk
Installing Photon is quite easy: [create a new VM](https://github.com/vmware/photon/wiki/Running-Project-Photon-on-vSphere) within vSphere and boot it with the downloaded Photon ISO file. The installation wizard will prompt you to select a micro, minimal or full installation. The minimal installation type already comes with all the functionality we need. Detailed setup instructions can be found in the [GitHub repository](https://github.com/vmware/photon/wiki/).

After the basic setup you can configure your Photon using [Cloud-Init and DNS](https://github.com/vmware/photon/blob/master/docs/cloud-init.md). The Bash commands shown in the listing below will provide you with an exemplary Cloud-Init configuration that's compatible with Photon. The result of the commands is a ISO image which you put into the virtual tray of your Photon VM at boot time. The image will configure your Photon's hostname and enable the Docker remote API, which is turned off by default. Thanks to the pre-installed VMware Tools you can grap the IP address, assigned using DHCP, from your vSphere client. Your Photon host is not ready to be used with any Docker client.

{% highlight Bash linenos %}
mkdir /tmp/photon-cloud-init/
cd /tmp/photon-cloud-init/

cat << EOF > user-data
#cloud-config
#Hostname of your photon host
hostname: myphotonhost
#Enable Docker remote API
runcmd:
 - sed -i -e '/^ExecStart/s/^.*$/ExecStart=\/bin\/docker -d -s overlay -H unix:\/\/\/var\/run\/docker.sock -H tcp:\/\/0.0.0.0:2375/' /lib/systemd/system/docker.service
 - systemctl daemon-reload
 - systemctl restart docker
EOF

cat << EOF > meta-data
instance-id: iid-local01
local-hostname: cloudimg
EOF

genisoimage -output seed.iso -volid cidata -joliet -rock user-data meta-data
{% endhighlight %}

#Integration paths
VMware is not only the name of the company behind the ESXi hypervisor. There's a huge arrangement of solutions built on top and around VMware's virtualization technology. A container runtime on it's own wouldn't be much of a container integration. But using various technologies within the VMware stack allows for providing Container-As-A-Service à la Amazon using your existing infrastructure. Therefore we'll be using products of the VMware vRealize product family: vRealize Orchestrator and vRealize Automation. If you only got a basic vSphere infrastructure at your hands: don't worry. Most of the examples shown below also work using the native integration of vRealize Orchestrator in the vSphere-Web-Client. The only requirement for this is a vSphere environment running in version 5.5.2 or above.

#vRealize Orchestrator
A special tool in the VMware tool belt comes with your vCenter license and therefore already exists in most IT departments, which makes it even more interesting. I'm talking about vRealize Orchestrator, which is a workflow engine that, using multiple plug-ins, is able to talk with many VMware and non-VMware products and allows for a huge degree of customization.

In order to implement Docker-As-A-Service we'll be using the open source plug-in Coopto I wrote some time ago. It enables your to manage containers in a uncomplicated way inside vRealize Orchestrator. Combined with other workflows and plug-ins you can build and automate your IT processes across multiple layers of your IT stack. For example you can create a workflow which will create a dedicated virtual hard disk for every container or dynamically create layer-2 networks for your containers using VMware NSX.

Before we start you'll need vRealize Orchestrator. You can download the latest version (6.x) as a virtual Appliance packed as OVA from VMware's website and [deploy it with little effort](http://pubs.vmware.com/vsphere-60/index.jsp?topic=%2Fcom.vmware.vrealize.orchestrator-install-config.doc%2FGUID-64F03876-2EAB-4DB3-A95D-89842425FF7A.html).
Next you'll have to download the Coopto plug-in from [VMware Solution Exchange](https://solutionexchange.vmware.com/store/products/coopto-a-docker-plug-in-for-vmware-vrealize-orchestrator--2#.VctI8fm2qmS) or from it's [GitHub repository](https://github.com/m451/coopto/). Within the Solution Exchange you'll also find the user-guide of the plug-in with detailed information how to install it. Once installed you can start the Java based Orchestrator client by visiting *https://orchestrator-host:8281/vco/* and clicking the *start orchestrator client* link.

##Configure the Docker plug-in
So you got Orchestrator running and the plug-in installed. Coopto with all of it's workflows should be visible in the client's workflow tab at *Library/Coopto*.

<figure>
  ![The Docker plug-in for vRealize Orchestrator appears at Library/Coopto](/assets/posts/2015-09-20-docker-as-a-service/plugin-workflows.png)
  <figcaption>Fig 2. The Docker plug-in for vRealize Orchestrator appears at Library/Coopto</figcaption>
</figure>

Now you got to add your Photon host as a Docker node. Right click the workflow *Library/Coopto/Configuration/Add Docker Node* and select *start workflow*. After you put in the host's IP address and the Docker remote API port you used in your Cloud-Init above (2375 by default) you can start the workflow by clicking *submit*. Please note that it's important that your vRealize Orchestrator host is able to connect to the Docker service on your Photon host and that you have to restart the Docker service within your Photon host after you turn on the remote API.

##Using the Docker plug-in
Deploying new containers consists of three steps: first you have to make the Docker image you want to use available on your Docker node. The workflow *Images/Pull Image* will help you with that. After starting the workflows you **select your Photon host** as the Docker node to use and press **next**. Now you enter the name of the [image](https://hub.docker.com/explore/) you like to use for example **busybox:latest**. Because this image is available in the official Docker hub your Photon host, in order to download the image, needs access to the internet. A little green check mark below the workflow you just ran should signal the successful execution.

The second step is to create a container from the image and is done using the *Containers/Create Container* workflow. You can leave most of the options to their defaults. For a meaningful Busybox container it's enough to select the image you just downloaded and start any Busybox process within the container. To do so you have to enter the command to run in the CMD input field, e.g. use */bin/ash* to open a shell in the Busybox container after you start it later on. If you want to connect to the container later on you also should enable the *attach stdin/stdout*, *open stdin* and *attach tty* options in the workflow.

The thrid and last step is to actually start the container. Run the *Containers/Start Container* workflow using the container created earlier as input object to do so. Leave all other options to their defaults and execute the workflow. If you now change to the inventory tab of your Orchestrator client you should find your Photon host and the container running inside of it listed below the Coopto plug-in.

<figure>
  ![The inventory tab shows the current Docker configuration](/assets/posts/2015-09-20-docker-as-a-service/plugin-inventory.png)
  <figcaption>Fig 3. The inventory tab shows the current Docker configuration</figcaption>
</figure>

There's no real value using this method to deploy containers just yet. Instead of a simple *docker run* command you have to go through three single configuration steps.
**It gets interesting once you understand how vRealize Orchestrator works and you start chaining all this single tasks into workflows and combine them with other plug-ins to implement the service you want to provide.**
The possibilities available here go far beyond what's possible with e.g. Docker compose and allow you to combine, amongst others, your vSphere-, networking- and storage-processes with containers.
If you look into the *Sandbox* folder within Coopto's workflows directory you'll find examples for creating such composed services. For example the workflow *Ceate Multi-Tier Liferay Service* shows how to build a service using multiple containers. Services you once defined like this can then be provided as single-click services to your users.

##Containers-As-A-Service
vRealize Orchestrator is a management tool to build any service, so called XaaS, in VMware. With a little help of plug-ins it's possible to build Docker-As-A-Service workflows this way. In order to provide those services to your users your best bet is VMware's self-service portal vRealize Automation. You can provide all services build in vRealize Orchestrator in vRealize Automation's web-portal and even add granular access rights and approval policies to those services. The component used there is the [Advanced Service Designer](http://pubs.vmware.com/vra-62/index.jsp?topic=%2Fcom.vmware.vra.asd.doc%2FGUID-EB8CCBE0-405D-43C9-9D12-209E56DD6CC0.html), which will trigger the Orchestrator workflows and request required parameters from the user.

<figure>
  ![Within vRealize Automation users can request containers and other services as-a-service](/assets/posts/2015-09-20-docker-as-a-service/vra-integration.png)
  <figcaption>Fig 4. Within vRealize Automation users can request containers and other services as-a-service</figcaption>
</figure>

If vRealize Automation is something you don't have yet you can also use the vSphere Web-Client to make workflows available for end users (althought, as you can imagine, the feature set is limited here). vSphere Administrators can then use the same UI to provision virtual machines as well as containers.

<figure>
  ![Workflows built in vRealize Orchestrator can also be executed from your vSphere Web-Client](/assets/posts/2015-09-20-docker-as-a-service/vcsa-integration.png)
  <figcaption>Fig 5. Workflows built in vRealize Orchestrator can also be executed from your vSphere Web-Client</figcaption>
</figure>

##Summary
Already today it's possible to deploy container services similar to Amazon within your own vSphere infrastructure. Additional products like vRealize Automation allow for easy provisioning using a web-portal. Besides Infrastructure-As-A-Service and Containers-As-A-Service any other service (XaaS) can be deployed as well. **The deal: your users get their containers exactly when they need them. Access to the Docker host itself however is left to the administrator. Because containers don't change the host configuration you get a straight separation of responsibilities between operations and development (or other departments) without loosing the advantage of Docker.**
By using your virtual hardware as a base for containers you also keep your existing infrastructure and can combine both technologies using the generic workflow engine vRealize Orchestrator.

Besides Photon VMware is working on other tools for even better container integration. The projects [Lightwave](https://github.com/vmware/lightwave) and [Bouneville](https://www.youtube.com/watch?v=q0Xg7mVOO8o) look very promising and soon it could be possible to control containers the same way you control virtual machines today.
It will be existing to see how things turn out and eventually it's not about containers versus virtual machines but more about a meaningful combination of them. Time will show how both of those technologies will eventually grow together.
