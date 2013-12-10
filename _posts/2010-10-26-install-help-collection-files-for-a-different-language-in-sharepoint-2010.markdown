---
author: Peter Gerritsen
comments: true
date: 2010-10-26 13:07:18+00:00
layout: post
slug: install-help-collection-files-for-a-different-language-in-sharepoint-2010
title: Install Help Collection files for a different language in SharePoint 2010
wordpress_id: 871
disqus_identifier: 871 http://blog.petergerritsen.nl/?p=871
tags:
- Language Packs
- SP2010
---

After installing a languagepack help contents should be available when you create a site collection in that language. 
Sometimes however SharePoint gives you a message that the files have not been installed when you click the help button or go to the Help-settings page for your sitecollection. 

You can force SharePoint to install the files using the hcinstal.exe tool in the bin folder under the SharePoint root (c:\program files\common files\microsoft shared\web server extensions\14\bin):

```
hcinstal.exe /act InstallAllHCs /loc 1043
```
This command will install all available help content for LCID 1043 (Dutch)

After this running this command (which can take up to 10-15 minutes easily) you need to run the SharePoint Products and Configuration Wizard on each front-end server.

I think you can also use the following 2 PowerShell commands, but I wasn't able to test these after the first solution worked:

  * Install-SPHelpCollection
  * Install-SPApplicationContent

