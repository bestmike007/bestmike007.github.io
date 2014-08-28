---
title: 'NetBeans: unnecessary deployment when tomcat starts'
author: mike
layout: post
permalink: /2010/09/netbeans-unnecessary-deployment-when-tomcat-starts/
jabber_published:
  - 1285247797
categories:
  - IT
tags:
  - NetBeans
---
Something annoying me when using NetBeans 6.9 as my development environment.

This bug has been reported to NetBeans Team. <http://netbeans.org/bugzilla/show_bug.cgi?id=190406>

<!--more-->Overview:

When I start to run my web site, tomcat will be started. And before deploying

my own application, all the formerly deployed sites will be started and

sometimes it takes a long time but it&#8217;s unnecessary. Why not delete all the

deployed applications before starting the tomcat?

Steps to Reproduce:

1. Create a web project and produce a web site which takes a long time to

initiate.

2. Deploy this new web site to tomcat.

3. Stop tomcat by clicking &#8220;Stop the Server&#8221; button in the output window.

4. Deploy the new web site again.

Actual Results:

1. Starting Tomcat.

2. Starting the deployed web site.

3. Undeploying the web site.

4. Deploying the web site agian.

Expected Results:

1. Starting Tomcat which is clean.

2. Deploying the web site to be run.

Build Date & Platform:

Product Version: NetBeans IDE 6.9 (Build 201006101454)

Java: 1.6.0_20; Java HotSpot(TM) 64-Bit Server VM 16.3-b01

System: Windows Server 2008 R2 version 6.1 running on amd64; GBK; en_us (nb)

Userdir: C:UsersAdministrator.netbeans6.9

Additional Information:

I don&#8217;t know whether it would happen on the other platform or not.

It&#8217;s not a critical bug but I think it&#8217;s a problem, because it always takes me

a long time before it&#8217;s deploying what I really need to deploy. Tomcat in

NetBeans is using a dedicated catalina base folder for development use, so I

think the deployed web apps should be removed in case the unnecessary waste of

time.

I hope that it could be solved as soon as possible.