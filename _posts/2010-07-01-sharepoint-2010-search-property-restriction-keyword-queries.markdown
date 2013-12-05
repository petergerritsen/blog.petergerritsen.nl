---
author: admin
comments: true
date: 2010-07-01 12:01:57+00:00
layout: post
slug: sharepoint-2010-search-property-restriction-keyword-queries
title: 'SharePoint 2010 Search: Property Restriction in Keyword Queries'
wordpress_id: 808
tags:
- Search
- SP2010
---

The new version of SharePoint offers more capabilities in the keyword syntax to enhance the search experience.
While in the previous version you had to resort to FullText SQL queries, a lot of things can now be accomplished with keyword syntax.

For instance if you would like to filter on a date, you could use the following query:
[code]LastModifiedTime&amp;gt;=01/06/2010[/code]

The actual format of the date depends on your regional settings (I'm using Dutch (nl-NL) in this case).

To search within a range of dates you can use the following query syntax:
[code]LastModifiedTime:28/06/2010..30/06/2010[/code]

To exclude items you could use the following syntax:
[code]LastModifiedTime<>28/06/2010[/code]

More info from MSDN about keyword syntax kan be found here: [http://msdn.microsoft.com/en-us/library/ee558911.aspx](http://msdn.microsoft.com/en-us/library/ee558911.aspx)
