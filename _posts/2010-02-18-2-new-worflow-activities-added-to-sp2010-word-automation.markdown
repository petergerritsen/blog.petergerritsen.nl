---
author: Peter Gerritsen
comments: true
date: 2010-02-18 19:56:16+00:00
layout: post
slug: 2-new-worflow-activities-added-to-sp2010-word-automation
title: 2 new worflow activities added to SP2010 Word Automation
wordpress_id: 746
tags:
- CodePlex
- SP2010
- Word Automation
---

I’ve added two new worfklow activities, Convert Folder and Convert Library, to the [SP2010 Word Automation project on CodePlex](http://sp2010wordautomation.codeplex.com).

Because you can’t associate workflows created with SharePoint designer to libraries or folders, these actions won’t use the current item from the context, so you need to specify the input and output library or folder by url. To use the activities you can run the workflow on a other item or document. The activities locate the libraries or folders relative to the current web, so you don’t have to specify a full url:

[![Convert Library Activity](/images/oldimage_thumb7.png)](/images/oldimage7.png)

[![Convert Folder Activity](/images/oldimage_thumb8.png)](/images/oldimage8.png)

You can download the latest release and source code from the [CodePlex project site](http://sp2010wordautomation.codeplex.com)
