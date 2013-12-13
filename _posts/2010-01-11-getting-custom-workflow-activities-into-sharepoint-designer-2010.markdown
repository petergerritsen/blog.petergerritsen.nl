---
author: Peter Gerritsen
comments: true
date: 2010-01-11 19:15:00+00:00
layout: post
slug: getting-custom-workflow-activities-into-sharepoint-designer-2010
title: Getting Custom Workflow Activities into SharePoint Designer 2010
wordpress_id: 652
tags:
- SharePoint Designer 2010
- SP2010
- Workflow
categories: SharePoint
---

Developing a custom workflow activity for SharePoint 2010 doesn’t differ that much from developing one for the MOSS 2007 platform. So by following the different articles on that you will be able to create one with ease. 

 

SharePoint 2010 still uses the same mechanism with an .ACTIONS file and adding an “authorizedType” element to your web.config. I was unsuccessful however in getting the activity to show up in SharePoint Designer 2010. 

 

After adding a “SafeControl” entry to the web.config the activity did show up. As far as I can see this is the only thing different to the steps you have to take in MOSS 2007.
