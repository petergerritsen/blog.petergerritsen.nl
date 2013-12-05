---
author: admin
comments: true
date: 2009-04-03 14:09:18+00:00
layout: post
slug: useful-sharepoint-classes
title: Useful SharePoint classes
wordpress_id: 553
tags:
- MOSS
---

I just found out that the object model includes some very useful classes to speed up your coding efforts.

**SPUtilty**

Contains methods for redirecting users to the error page, access denied page or a custom url. You can get Full name and email adres of a user by passing in the logginname, send an email from the web context or determine if an lcid is an East-Asian lcid.

More information here:
[http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.utilities.sputility.aspx](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.utilities.sputility.aspx)

**SPBuiltinFieldId**

Contains variables for all the Guids of the built in fields. No need to worry about the difference between dutch and english MOSS sites.

**SPBuiltInContentTypeId**

Contains variables for all the Guids of the built-in contenttypes.

**SPDiffUtilty**

Shows the differences between two strings in Html format. So comparing “This is an initial string” with “This” returns the following: “**This is an initial string**“

**SPContentTypeId (structure)**

Provides methods to determine the relationships between two contenttypes.

**SPContentTypeUsage**

Allows you to determine where contenttypes are used within the sitecollection. Here’s a useful post that shows code to audit the contenttype hierarchy:
[http://soerennielsen.wordpress.com/2008/03/06/audit-your-content-type-hierarchy/](http://soerennielsen.wordpress.com/2008/03/06/audit-your-content-type-hierarchy/)

**SPChangeQuery**

Allows you to query your sitecollection for objects that have changed. This way you can audit changes in access rights for instance.
