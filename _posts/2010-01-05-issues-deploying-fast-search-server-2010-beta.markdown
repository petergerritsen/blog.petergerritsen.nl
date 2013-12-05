---
author: admin
comments: true
date: 2010-01-05 13:50:00+00:00
layout: post
slug: issues-deploying-fast-search-server-2010-beta
title: Issues deploying FAST Search Server 2010 Beta
wordpress_id: 668
tags:
- FAST 2010
- SP2010
---

I just had a tough time deploy FAST Search Server 2010 Beta on a new SharePoint 2010 farm. Upon searching the internet it looked like I had the same issue as loads of other people, a not complete/wrong installation guide. But even after reviewing the posts in [this](http://social.technet.microsoft.com/Forums/en-ZA/sharepoint2010setup/thread/f653c63c-34ff-4215-bfbc-17d3d26bd6c9) thread and reading the [post](http://blogs.msdn.com/mberry/archive/2009/12/04/configuring-sharepoint-2010-for-fast-search-server-query-and-admin.aspx) from Manfred Berry, I was unsuccessful in getting FAST to work. 

 

Until I dove into the logs on the FAST server, which is something I always postpone due to the overload of information in there. I found an error mentioning “Unrecognized attribute 'allowInsecureTransport'”, caused by the dreaded WCF issue that needs the same hotfix as metioned [here](http://blogs.msdn.com/sharepoint/archive/2009/11/19/installation-notice-for-the-sharepoint-server-public-beta-on-microsoft-windows-server-2008-r2-and-microsoft-windows-7.aspx). So not only install the hotfix on your SharePoint servers but also on your FAST servers, which seems kind of logical now I think of that.
