---
author: Peter Gerritsen
comments: true
date: 2010-02-11 19:32:03+00:00
layout: post
slug: sp2010-installation-error-creating-configuration-database
title: SP2010 Installation - Error creating configuration database
wordpress_id: 740
tags:
- SP2010
categories: SharePoint
---

When I tried to install the new RC of SharePoint 2010 on my machine, I got an "Error creating configuration database" message. When I went to the installation log I found a "User cannot be found" error. The cause was that the configuration wizard could not find the AD controller, which was easily solved by opening a VPN connection as I was working from home.
