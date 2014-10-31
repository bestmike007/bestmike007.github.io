---
layout: post
title: "Applying Patches on Unsupported XenServer 6.2"
date: 2014-10-31 07:56:41 +0000
comments: true
categories: 
  - IT
tags:
  - XenServer
---
It's easy to patch hostfixes through official XenCenter management software for XenServers with license, while without a license the XenServer have to be patched manually. Official website has an article on [how to apply a XenServer hotfix manually using the command line](http://support.citrix.com/article/CTX132791L). And this process is able to be done automatically with some scripts.

<!--more-->

## How to Patch

Usually we use XenCenter to manage our standalone XenServers and XenServer clusters. XenCenter is checking for updates when connected to a XenServer (or pool master). It will show up like the following image.

{% img /uploads/2014-10-31-applying-patches-on-unsupported-xenserver-6-dot-2/xencenter-update-notification.jpg %}

By clicking the 'Go to the Web Page...' button, the browser will open a web page to a support article introducing the details about the hotfix patch. And we are able to get the download link to the patch package after opening the 'Download Manager' page.

{% img /uploads/2014-10-31-applying-patches-on-unsupported-xenserver-6-dot-2/citrix-download-manager.jpg %}

And we'll get a download like this: ```http://downloadns.citrix.com.edgesuite.net/akdlm/*/XS*.zip```. With this link we can patch our XenServers by following the steps described in the [official article](http://support.citrix.com/article/CTX132791L).

## The Script

We usually open the server (pool master if not standalone installation) console and enter several commands and copy & paste intermediate outputs to apply the patch. So I have simplified the process by the following script.

<pre>
#!/bin/bash
# install a update package from citrix
# e.g. http://downloadns.citrix.com.edgesuite.net/akdlm/*/XS*.zip
URL=$1
PKG=`basename $URL`
cd /tmp
mkdir update
cd update
wget -O $PKG $URL
unzip $PKG
for file in $(ls *.xsupdate); do ( xe patch-pool-apply uuid=`xe patch-upload file-name=$file` ) done
cd /tmp
rm -rf update
</pre>

We can save this script to ```/root/patch```. And with this script, we simply apply the patch by running a single command.

<pre>bash# ./patch http://downloadns.citrix.com.edgesuite.net/akdlm/*/XS*.zip</pre>

## Patch the Server Automatically

You might be thinking that there have to be some update feed for XenCenter to check for updates, with which we are able to patch the server automatically. That's right. I found this tool: [citrix_xenserver_patcher](https://github.com/dalgibbard/citrix_xenserver_patcher). This tool use the [patch feed](http://updates.xensource.com/XenServer/updates.xml) to check for updates and patch the server automatically.

And further more, we can also use cron jobs to patch the server silently. However I prefer patching the server with awareness in case of exceptions.

## How to Reboot after Patching XenServer Pool

If we're patching a standalone XenServer, we need to stop or pause all the running VMs before rebooting the XenServer if reboot is required after the patch; while we're patching a XenServer pool, running VMs can be migrated to perform a sequential reboot without interrupting running services. Toolstacks of different versions might not be able to communicate correctly, for that we can proceed with the following steps:

1. Restart toolstack on non-pool-master servers
2. Restart toolstack on pool master
3. Migrate VMs and reboot non-pool-master servers sequentially
4. Migrate VMs and reboot the pool master