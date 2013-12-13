---
author: Peter Gerritsen
comments: true
date: 2010-01-13 14:27:45+00:00
layout: post
slug: codeplex-project-for-word-automation-services
title: CodePlex project for Word Automation Services
wordpress_id: 532
tags:
- CodePlex
- SP2010
- Word Automation
categories: SharePoint
---


I’ve just published the first release for a [CodePlex project](http://sp2010wordautomation.codeplex.com)  I started to provide sample projects / solutions for using the Word Automation Services in SharePoint 2010.






Word Automation Services allow you to convert document to and from different formats.






_File formats the service can read:_

  * 
_Office Open XML (DOCX, DOCM, DOTX, DOTM) _
  *
_Word 97-2003 Document (DOC) and Word 97-2003 Template (DOT)_
  *
_Rich Text Format (RTF) _
  *
_Single File Web Page (MHTML) _
  *
_HTML _
  *
_Word 2003 XML _
  *
_Word 2007/2010 XML_

_File formats the service can write:_

  *
_PDF _
  *
_XPS _
  *
_Office Open XML (DOCX, DOCM) _
  *
_Word 97-2003 Document (DOC) _
  *
_Rich Text Format (RTF) _
  *
_Single File Web Page (MHTML) _
  *
_Word 2007/2010 XML_

([source](http://blogs.msdn.com/microsoft_office_word/archive/2009/12/16/Word-Automation-Services_3A00_-What-It-Does.aspx))

As far as I’ve found out, there are no UI features available out-of-the-box to use these services, so I’ve decided to create some. The first one is a custom workflow action you can use in SharePoint Designer to convert a document to many of the supported
formats.


In the workflow designer you can add the “Convert Document” action:

[![Workflow Actions](/images/old/snipping1.png)](/images/old/snipping.png)

The action is inserted into the workflow step where you can specify the url of the output file, select the output format and save options and select a variable for storing the conversion job id (which you can use later to retrieve the status, as the job runs asynchronous):

[![Convert document action](/images/old/snipping3.png)](/images/old/snipping2.png)

[![Save Behaviour](/images/old/snipping5.png)](/images/old/snipping4.png)

The job id is also logged into the Workflow History Log (the second entry is from a second workflow action that logs the returned conversion job id variable):

[![image](/images/old/snipping7.png)](/images/old/snipping6.png)

After the job has run, which can take up to a few minutes (depending on the word automation services settings), the converted  document appears in the library:

[![image](/images/old/snipping9.png)](/images/old/snipping8.png)

The custom workflow action is one of the first features for Word Automation in SharePoint 2010 I’ve planned to release. Other  features will be a Ribbon and Item context menu extension and more Workflow actions.

Let me know if you have any suggestions for improvement or other functionality you would like to see.

