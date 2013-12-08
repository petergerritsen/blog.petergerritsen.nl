---
author: Peter Gerritsen
comments: true
date: 2010-07-12 08:47:15+00:00
layout: post
slug: css-contents-showing-in-settings-page-for-site-based-on-custom-template
title: CSS contents showing in settings page for site based on custom template
wordpress_id: 816
tags:
- SP2010
---

At a customer we created a few custom site templates by configuring them and then saving them as template. When creating new sites based on this template, we had the strange issue that the contents of the Alternate CSS (AlternateCSSUrl) were included and showing in the header on layouts pages in the sites.
It appears that the AlternateCSSUrl is also set on the AlternateHeader property of the SPWeb object.

This [post](http://social.technet.microsoft.com/Forums/en/sharepoint2010customization/thread/599d42d4-72c0-4688-af52-91fb7528fe60) on the SharePoint 2010 forums also mentions this issue.

You can off course use powershell to get rid of this problem:

```
$site = Get-SPSite "http://sitecollectionurl"
$web = $site.OpenWeb("/weburl")
$web.AlternateHeader = ""
$web.Update()
```

Update: 
The post on the forums got answered and according to that it has to do with the publishing feature that was activated before saving the site as a template. However in my case that feature wasn't activated, because we're aware that such a scenario isn't officially supported by Microsoft. 
