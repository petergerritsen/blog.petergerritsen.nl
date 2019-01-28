---
title: 'ARM: Whitelist outbound IP''s from another app service '
layout: post
comments: true
author: Peter
date: 2019-01-28 12:37:21 +0000
slug: arm-whitelist-outbound-ips-from-another-app-service
wordpress_id: ''
tags:
- Azure Resource Manager
- Azure Networking
categories:
- Azure

---
In a [previous post](/2019/01/28/arm-create-generic-ip-whitelisting-template) I wrote about adding one or more specified ranges of IP addresses to the IP security restrictions of an App Service.

In this post, I would like to take it one step further: add the possible outbound IP addresses of another App Service to the white list.

**_N.B._** _Securing a web app in this way is not a total security solution, because App Services of other Azure customers can share the same outbound IP addresses within the shared network infrastructure. If that's a requirement, you will need to resort to the isolated App Service Environment, but that's in a different pricing level._

But how do you do that? You can off course get the outbound IP addresses from another App Service by using the reference function within a template.  But this is a comma separated list of addresses. So you will need to split this into an array and add the subnet mask (in CIDR notation) and action properties to use it as a property in the sites/config/ipSecurityRestrictions element I've described in a previous post. Things get more difficult when you realize you can only use the reference function in the resources of output section of a template. So there's no way you can create a variable that gets the list (as a comma separated string) and splits it in to an array you can use further on in the template when deploying resources.

Enter multiple LinkedTemplates and a bit of useful information:

_A template doesn't need to deploy a resource_

So this means you can create a template, that takes a resourceId for an App Service, gets the range of possible outbound IPs and returns it as an array:

<script src="[https://gist.github.com/petergerritsen/cfea06a3936b77d1d1907ca4e2bb4a2f.js](https://gist.github.com/petergerritsen/cfea06a3936b77d1d1907ca4e2bb4a2f.js "https://gist.github.com/petergerritsen/cfea06a3936b77d1d1907ca4e2bb4a2f.js")"></script>

But this doesn't create an object array which we can pass to the template that adds the ranges to an App Service as described in the previous post.

So we create another template that takes an array of strings with the different IP's and returns an array with the object in the required form:

<script src="[https://gist.github.com/petergerritsen/17f68f16411efa5236b95d6c6306bda3.js](https://gist.github.com/petergerritsen/17f68f16411efa5236b95d6c6306bda3.js "https://gist.github.com/petergerritsen/17f68f16411efa5236b95d6c6306bda3.js")"></script>

We can then modify the first template to call this template and instead return the result of that template:

<script src="[https://gist.github.com/petergerritsen/3f5866a19244cb7ddcd7b1b2a917cbe3.js](https://gist.github.com/petergerritsen/3f5866a19244cb7ddcd7b1b2a917cbe3.js "https://gist.github.com/petergerritsen/3f5866a19244cb7ddcd7b1b2a917cbe3.js")"></script>

All that remains now, is to call this template from the main template that deploys the App Service and pass this to the template that actually adds the ranges to the whitelist: