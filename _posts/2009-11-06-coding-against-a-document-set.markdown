---
author: Peter Gerritsen
comments: true
date: 2009-11-06 16:56:31+00:00
layout: post
slug: coding-against-a-document-set
title: Coding against a Document Set
wordpress_id: 541
disqus_identifier: 541 http://blog.petergerritsen.nl/2009/11/06/sp2010-coding-against-a-document-set/
tags:
- Document Sets
- Object Model
- SP2010
categories: SharePoint
---

I’ve just had a 5 day training on development for SharePoint 2010. We’ve seen some cool stuff, were able to do some Hands on Labs and talk to other SharePoint experts about the new stuff that is coming up.

One of the best things though about this training was the ability to try some stuff out for yourself. The fact that no project managers or customers were bothering me for a week allowed me to finally take the time to look around in the object model, the central admin and the new front end interfaces.

Over the next few week I’ll be publishing some post on the things I’ve tried out.I hope to publish a lot more posts about SP2010 once Beta 2 comes available.

DISCLAIMER: The examples are build on and tested against a Beta 1 build of SharePoint 2010 and a Beta 1 build of Visual Studio 2010, so there is no guarantee this will work on later versions.

One of the things I was eager to try out is the new Document Sets feature. Document Sets allow you to group related documents together and share metadata between those docs. When you view a document set in a library, you are presented with a welcome page that shows the metadata of the set and the contents. The welcome page in itself is something you’re able to customize. So you can add web parts and other controls through the interface or SharePoint Designer to the welcome page.

First let me start by giving you some code you can use to show extra information about
the document set in a web part you can place on the welcome page:

```csharp
try
{
     SPListItem item = SPContext.Current.ListItem;
     DocumentSet set = DocumentSet.GetDocumentSet(item.Folder);
     writer.WriteLine("ContentType: {0}<br/>", item.ContentType.Name);
     writer.WriteLine("Title: {0}<br/>", item.Title);
     writer.WriteLine("WelcomePageUrl: {0}<br/>", set.WelcomePageUrl);
     writer.WriteLine("ItemCount: {0}<br/>", set.Folder.ItemCount);
     writer.WriteLine("Welcomepage Fields:<br/>");
     DocumentSetTemplate template = set.ContentTypeTemplate;
     WelcomePageFieldCollection fields = template.WelcomePageFields;
     foreach (SPField field in fields)
     {
         writer.WriteLine("{0}<br/>", field.Title);
     }
}
catch (Exception)
{ }
```

First we get a reference to the current List Item through the SPContext. This list item is the main item for the document set that contains the metadata that is pushed into the child documents.

We then can get a reference to the DocumentSet by passing in the SPFolder of the item into a static method of the DocumentSet class. The DocumentSet class is stored in the Microsoft.Office.DocumentManagement.dll in the DocumentSets namespace.

The DocumentSetTemplate in turn contains more information about the fields that are shared or shown on the Welcome page.

In the next post I’ll show you how to provision a document set from a feature. Something that is quite easy to do with the new SharePoint project and item templates for Visual Studio 2010.
