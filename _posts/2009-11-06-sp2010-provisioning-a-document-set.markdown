---
author: admin
comments: true
date: 2009-11-06 17:32:27+00:00
layout: post
slug: sp2010-provisioning-a-document-set
title: Provisioning a Document Set
wordpress_id: 540
tags:
- Document Sets
- SP2010
---

In this post I’ll show you how to create a project in Visual Studio 2010 with the new SharePoint project and item templates to provision a Document Set from a feature.

DISCLAIMER: The examples are build on and tested against a Beta 1 build of SharePoint 2010 and a Beta 1 build of Visual Studio 2010, so there is no guarantee this will work on later versions or even on the Beta 1 build you're running.

A Document Set is basically a content type just like all the others you can find in SharePoint, it derives from the Folder content type. So the steps you need to take to provision a Document Set content type are not that different as well.

We’ll start out by creating an empty SharePoint project in Visual Studio 2010 and work from there.

First we’ll add a Content Type item to the project. In the elements.xml file we place the following content:
[sourcecode="xml"]











DocumentSet ItemUpdated
Synchronous
10002
100
Microsoft.Office.DocumentManagement, Version=14.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c
Microsoft.Office.DocumentManagement.DocumentSets.DocumentSetEventReceiver




DocumentSet ItemAdded
Synchronous
10001
100
Microsoft.Office.DocumentManagement, Version=14.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c
Microsoft.Office.DocumentManagement.DocumentSets.DocumentSetItemsEventReceiver





























DocSetDisplayForm
ListForm
DocSetDisplayForm




_layouts/NewDocSet.aspx





[/sourcecode]
As you can see, the basics are the same as for any content type. The main difference is in all the XmlDocument elements in there:



	
  * Some event handlers are hooked up to make sure the metadata gets pushed down into the child documents (plus some other stuff)

	
  * The content types that users are allowed to add to the set are specified

	
  * We specify which fields are shared between the documents the set contains

	
  * The fields that are shown on the welcome page are defined as well

	
  * We then specify if there’s default content to add when a new Document Set is created


After we’ve created the basic plumbing for the Document Set content type, we’ll need to make sure that the files that are required are created in the right place as well. In order to accomplish this we’ll add a SharePoint Module item to the solution. This module will create the welcome page and default content in the right location in the site collection. The element.xml file will contain the following content:

[sourcecode="xml"]


















[/sourcecode]

We see that the page layout for the document set homepage is created in the _cts folder for the content type. The web parts that are placed on the page are configured here as well, so any modifications and additions will be used on the welcome page of all document sets based in this content type. Also the two documents for the default content are placed in the corresponding _cts folder in the site collection.

The final Visual Studio solution will look like this:

[![image](http://blog.petergerritsen.nl/wp-content/uploads/snipping11.png)](http://blog.petergerritsen.nl/wp-content/uploads/snipping10.png)

After deploying the solution and activating the feature, which is very easy to do with the new SharePoint stuff in Visual Studio (just press ctrl + f5), we can see that the _cts folder will be created in the site collection:

[![image](http://blog.petergerritsen.nl/wp-content/uploads/snipping13.png)](http://blog.petergerritsen.nl/wp-content/uploads/snipping12.png)

After we add the content type to a document library and create a new item based on the content type we’ll be presented with the following:

[![image](http://blog.petergerritsen.nl/wp-content/uploads/snipping15.png)](http://blog.petergerritsen.nl/wp-content/uploads/snipping14.png)

You can download the sample solution here: [DocSetProvisioning.zip (62,88 KB)](http://blog.petergerritsen.nl/wp-content/uploads/DocSetProvisioning.zip)

DISCLAIMER: This hasn't been properly tested, so there's is no guarantee it will work. If it f***s up your farm, the most you can expect as support from me, is an email wishing you good luck with restoring it.
