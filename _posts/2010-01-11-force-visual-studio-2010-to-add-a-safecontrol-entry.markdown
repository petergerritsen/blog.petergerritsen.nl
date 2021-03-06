---
author: Peter Gerritsen
comments: true
date: 2010-01-11 19:44:00+00:00
layout: post
slug: force-visual-studio-2010-to-add-a-safecontrol-entry
title: Force Visual Studio 2010 to add a SafeControl Entry
wordpress_id: 650
tags:
- SP2010
- Visual Studio 2010
categories: SharePoint
---

When you create a project in Visual Studio 2010 on one of the SharePoint project templates it will take care of all the packaging for you. 

But when I was working on a project with custom workflow actions, the [SafeControl entry that is needed for making it work](http://blog.petergerritsen.nl/2010/01/11/getting-custom-workflow-activities-into-sharepoint-designer-2010/) was not added to the generated manifest.xml file. 

Fortunately the package designer allows you to modify the template file it uses for generating this file. So open up the package designer, switch to the “Manifest” tab and add the assembly reference in the template yourself, but this time, include the SafeControl entry: 

![Package definition](/images/old/image_2.png)

You can safely use the SharePoint project tokens in there as well, but only for the SafeControl entry. When you put it into the Assembly entry, the package generator won’t understand it and will add another assembly reference for the project output:

![Package definition](/images/old/image_4.png)
