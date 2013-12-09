---
author: Peter Gerritsen
comments: true
date: 2010-06-09 12:30:10+00:00
layout: post
slug: term-store-management-option-missing-in-site-collection-settings
title: Term Store management option missing in Site Collection settings
wordpress_id: 800
tags:
- Managed Metadata
- SP2010
---

When you don't see the Term Store Management option in your site collection settings:

[![](/images/old2010/06/Term-Store-Management-option-300x224.png)](/images/old2010/06/Term-Store-Management-option.png)

A hidden web application feature is probably not activated. You can activate it through PowerShell or stsadm:

```
Enable-SPFeature -id "73EF14B1-13A9-416b-A9B5-ECECA2B0604C" -Url <Site-URL>
```

```
stsadm -o activatefeature -id 73EF14B1-13A9-416b-A9B5-ECECA2B0604C -url http://<url> -force
```