---
layout: post
title: "HandsOn: Veeam - Backup-as-a-Service in vRealize Automation"
date: 2015-07-02 08:00:00
comments: true
tags: [author-chk]
author:
    name: "Christian Klose"
    twitter: "christianklose"
description: "Last week i prepared a use case for some customer presentations about Backup as a Service using Veeam B&R v8. I know about the Veeam REST API which is available for Version 8 with Veeam Enterprise Manager. I also know about the Powershell implementations but hey - i love VMware so i build solutions in Orchestrator :-) and decided to test drive some use case workflows in vRealize Orchestrator and also bind them to vRealize Automation."
---

This is a use case description for some customer presentations about Backup as a Service using [Veeam B&R v8](http://www.veeam.com/de/vm-backup-recovery-replication-software.html). This article was originally posted January 2015 on [my311.de](http://my311.de/).

#Overview

The main idea behind that is to enable VMware Administrators or Users directly to choose a Backup SLA for their VM without the need to communicate with the Backup team - sorry Backup teams :-) The Backup Team acts as a Service Provider and offers different SLA's which could be used by VMware Admins or Users.

I know about the Veeam REST API which is availbable for Version 8 with Veeam Enterprise Manager. I also know about the Powershell implementations but hey - i love VMware so i build solutions in Orchestrator :-) and decided to test drive some use case workflows in vRealize Orchestrator and also bind them to vRealize Automation.

# REST Host and vRealize Automation 6.2 built-in Orchestrator

When you tried to implement a REST Solution to vRealize Automation 6.2 with built-in Orchestrator you maybe know that there is a little problem with the Apache HTTP client. If not - read the Community article [here](https://communities.vmware.com/message/2463764#2463764). So i went with my vRO Appliance (5.5.2) which works fine for that use case.

For any information about Veeam REST API please have a look at the [Veeam Help Center](http://helpcenter.veeam.com/backup/80/rest/).


##### UPDATE 26.02.2015 :

There is a technical preview version of a new [REST plug-in Version 1.0.6](https://communities.vmware.com/docs/DOC-28670) available which i integrated to my vRA Appliance 6.2 with built-in Orchestrator v6. That fixes the Apache HTTP client issue and i was able to fully implement my solution to vRA 6.2 with integrated vRO.



# Add REST Host

There are several informations and blogs on how to add a REST Host to your environment - i just wanted to name [vcoportal.de](http://www.vcoportal.de) which is a great resource if you are using vRealize Orchestrator.

But adding a REST Host is - if everything works fine - as easy as these screenshots:

In vRealize Orchestrator Client navigate to *Workflows - Library - HTTP-REST - Configuration - Add a REST host* and start the workflow.


  ![HandsOn: Veeam - Backup-as-a-Service in vRealize Automation](http://www.vratpack.com/assets/posts/2015-07-02-veeam-baas/my311de-15.01.2015-13.52.10.png)

Please verify you're Veeam Hosts URL and REST Port. By default HTTP protocol port 9399 is used and for HTTPS protocol, port 9398 is used.

I don't use any Proxy so i "skip" Proxy Settings...


  ![HandsOn: Veeam - Backup-as-a-Service in vRealize Automation](http://www.vratpack.com/assets/posts/2015-07-02-veeam-baas/my311de-15.01.2015-13.51.33.png)

Host authentication is set to Basic with Shared Session mode and i used the Service Account configured for Veeam.



# Workflows and Actions

For my sample use case i created two simple workflows :

- Add a VM to an existing Backup Job
- Restore VM from Restore Point

The main idea behind them is to browse the *BackupJobs* and *RestorePoints* directly by using the **Predefined list of elements Property** in the workflow presentation and let the user or administrator directly choose the BackupJob or RestorePoint which he needs for his VM.


### Add VM to Backup Job

Add VM to Backup Job allows you to add a VM (VC:VirtualMachine) to an existing Backup Job.

  ![HandsOn: Veeam - Backup-as-a-Service in vRealize Automation](http://www.vratpack.com/assets/posts/2015-07-02-veeam-baas/my311de-19.01.2015-17.27.16.png)

At first select the VM in the Workflow or it maybe comes as input variable from vRealize Automation or as contextual menu from vRealize Orchestrator; but later more to both implementations.

  ![HandsOn: Veeam - Backup-as-a-Service in vRealize Automation](http://www.vratpack.com/assets/posts/2015-07-02-veeam-baas/my311de-19.01.2015-17.27.23.png)

In this BackupJob section i can't see any backup jobs in the dropdown list **SelectJob** because the search field **SearchJob** is empty.

  ![HandsOn: Veeam - Backup-as-a-Service in vRealize Automation](http://www.vratpack.com/assets/posts/2015-07-02-veeam-baas/my311de-19.01.2015-17.27.40.png)

When i start typing a known backup job name (case sensitive) the dropdown list fills up with the backup jobs matching the search string and select the backup job to which i could add the VM.

  ![HandsOn: Veeam - Backup-as-a-Service in vRealize Automation](http://www.vratpack.com/assets/posts/2015-07-02-veeam-baas/my311de-15.01.2015-14.43.29.png)

The predefined list of available Backup Jobs by Filter comes from the Presentation configuration in my Workflow. I configured the **SelectJob** Input variable with the Predefined list of elements with a connection to an action which delivers me the Backupjobs in an array from a defined Search String.

{% highlight javascript linenos %}
  GetAction("de.fum.veeam","getBackupJobs").call("#veeamHost,#SearchJob")
{% endhighlight %}



As an example here the Script for Action **getBackupJobs**

Input
 - veeamHost : *REST:RESTHOST*
 - SearchJob : *String*

Return type : *Array/String*

{% highlight javascript linenos %}

  var postResponse = veeamHost.createRequest("POST", "/api/sessionMngr/", null).execute();
  var backupJobResponse = veeamHost.createRequest("GET", "/api/jobs?type=job", null).execute();
  var xmlString = XMLManager.fromString(backupJobResponse.contentAsString);
  var xmlElement = xmlString.documentElement.getElementsByTagName("Ref");

  var backupJobs = [];

  for( i = 0; i < xmlElement.getLength(); i++ ){
    var backupJobName = xmlElement.item(i).getAttribute("Name")
    if ( backupJobName.indexOf(SearchJob) >= 0 ) {
    backupJobs.push(BackupJobName);
    }
  }
  return backupJobs;
{% endhighlight %}

After selecting VM (VC:VirtualMachine) and BackupJob (String) the Workflow does some converts to add the VM Name to the BackupJob ID by using [POST /api/jobs/{jobID}/includes](http://helpcenter.veeam.com/backup/80/rest/post_jobs_id_includes.html).

### Restore VM from Backup/RestorePoint

Restoring a VM brings you to one problem - what happens with an existing VM? In my first draft of the use case the answer is easy : i delete the existing VM and restore it from Backup (potential for improvements!!!).

  ![HandsOn: Veeam - Backup-as-a-Service in vRealize Automation](http://www.vratpack.com/assets/posts/2015-07-02-veeam-baas/my311de-19.01.2015-17.53.34.png)

I implemented a simple decision : does the VM still exist in vCenter Inventory and i want to restore it or has the VM been deleted and i want to restore it by using VM Name.

  ![HandsOn: Veeam - Backup-as-a-Service in vRealize Automation](http://www.vratpack.com/assets/posts/2015-07-02-veeam-baas/my311de-19.01.2015-17.53.47.png)

Selecting a VM from Inventory or typing the name to the field vmName returns the restorePoints as a Dropdown list where you can select the state you wanted to restore.


  ![HandsOn: Veeam - Backup-as-a-Service in vRealize Automation](http://www.vratpack.com/assets/posts/2015-07-02-veeam-baas/my311de-15.01.2015-14.44.23.png)

This is also solved by using the **Predefined list of elements** and an action that delivers exactly these RestorePoints.

{% highlight javascript linenos %}
    GetAction("de.fum.veeam","getRestorePoints").call("#veeamHost,#vm,#vmName")
{% endhighlight %}


# Mapping into vRealize Automation

Bringing both Worflows to vRealize Automation through **Advanced Services - Resource Actions** gives the User direct access to add his virtual machine to a backup job or restore it from a backup job.

Think about a pre-defined list of backup jobs for as-a-Service offerings in Veeam ("CLOUD") and pre-configure the SearchJob field in vRealize Automation with this ("CLOUD") filter name gives the User only the pre-defined list of jobs where he can the right service for his VM.

  ![HandsOn: Veeam - Backup-as-a-Service in vRealize Automation](http://www.vratpack.com/assets/posts/2015-07-02-veeam-baas/my311de-19.01.2015-17.51.52.png)
