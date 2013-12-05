---
author: admin
comments: true
date: 2010-09-20 11:55:56+00:00
layout: post
slug: strange-fulltextsqlquery-query-behaviour-system-servicemodel-faultexception
title: Strange FullTextSqlQuery query behaviour (System.ServiceModel.FaultException)
wordpress_id: 841
tags:
- Search
- SP2010
---

Today I was trying to fix an error in a custom webpart that performs a query on the SharePoint farm. The goal was to retreive the records within a site though the search system.

First I added a metadata property ItemDeclaredRecord that is mapped to ows__vti_ItemDeclaredRecord (DateTime). 
Then I tried the following query (using [Steve Peschka's Developer Search Tool](http://blogs.technet.com/b/speschka/archive/2010/08/15/free-developer-search-tool-for-sharepoint-2010-search-and-fast-search-for-sharepoint.aspx)):

`SELECT Size, Rank, Path, Title, Description, Write FROM scope() WHERE  (ItemDeclaredRecord > '1900/01/01 00:00:00')  AND CONTAINS (Path, '"/projecten"')
`

This query works fine and retreives all records declared after January 1st, 1900 under the "projecten" site. To retreive all records within a specific subsite I then tried the following query:

`SELECT Size, Rank, Path, Title, Description, Write FROM scope() WHERE  (ItemDeclaredRecord > '1900/01/01 00:00:00')  AND CONTAINS (Path, '"/projecten/subsite1"')`

This threw a FaultException?!
After fiddling around with the query for a few hours, I finally found a query that works:

`SELECT Size, Rank, Path, Title, Description, Write FROM scope() WHERE  (ItemDeclaredRecord > '1900/01/01 00:00:00')  AND CONTAINS (Path, '"/projecten/subsite1"') ORDER BY Rank DESC`

I'm not sure what the ORDER BY part does to the search system internally, but for now I'm guessing it's SQL for 'Pretty pretty please!'.

EDIT:
The property you add in the ORDER BY part has to have the "Reduce storage requirements for text properties by using a hash for comparison” option checked.
