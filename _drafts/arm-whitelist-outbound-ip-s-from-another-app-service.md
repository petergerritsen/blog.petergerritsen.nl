---
title: 'ARM: Whitelist outbound IP''s from another app service '
layout: post
comments: true
author: Peter
date: 2018-12-10 12:37:21 +0000
slug: ''
wordpress_id: ''
tags: []
categories: []

---
In a previous post I wrote about adding one or more specified ranges of IP addresses to the IP security restrictions of an App Service. 

In this post, I would like to take it one step further: add the possible outbound IP addresses of another App Service to the white list.

**_N.B._** _Securing a web app in this way is not a total security solution, because App Services of other Azure customers can share the same outbound IP addresses within the shared network infrastructure. If that's a requirement, you will need to resort to the isolated App Service Environment, but that's in a different pricing level._ 

But how do you do that? You can off course get the outbound IP addresses from another App Service by using the reference function within a template.  But this is a comma separated list of addresses. So you will need to split this into an array and add the subnet mask (in CIDR notation) and action properties to use it as a property in the sites/config/ipSecurityRestrictions element I've described in a previous post. Things get more difficult when you realize you can only use the reference function in the resources of output section of a template. So there's no way you can create a variable that gets the list (as a comma separated string) and splits it in to an array you can use further on in the template when deploying resources. 

Enter multiple LinkedTemplates and a bit of useful information: 

_A template doesn't need to deploy a resource_ 

So this means you can create a template, that takes a resourceId for an App Service, gets the range of possible outbound IPs and returns it as an array:

<script src="[https://gist.github.com/petergerritsen/cfea06a3936b77d1d1907ca4e2bb4a2f.js](https://gist.github.com/petergerritsen/cfea06a3936b77d1d1907ca4e2bb4a2f.js "https://gist.github.com/petergerritsen/cfea06a3936b77d1d1907ca4e2bb4a2f.js")"></script>