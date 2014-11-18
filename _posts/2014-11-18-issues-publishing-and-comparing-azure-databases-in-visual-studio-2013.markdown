---
author: Peter Gerritsen
comments: true
date: 2014-11-18 11:00:00+00:00
layout: post
slug: issues-publishing-and-comparing-azure-databases-in-visual-studio-2013
title: Issues publishing and comparing Azure Databases in Visual Studio 2013
tags:
- VS2013
- Azure SQL
categories: Azure
---

I was trying to move some Azure SQL databases to another subscription and had issues doing a publish of a SSDT project and also performing a schema compare from Visual Studio 2013.
When looking in the error list I saw it had to do with timeouts on the connection.

Updating the connection timeout in the connection properties didn't resolve the problem.

After googling / binging some more I discovered the following stackoverflow post:
[http://stackoverflow.com/questions/26070464/visual-studio-2013-publish-database-to-azure]

Appearantly usign a Basic Tier will give you these problems. After scaling up to Standard the issues were resolved. I have yet to find out if you can switch back to Basic after an initial setup of the DB and from there small incremental changes don't give issues.

