---
title: 'ARM: Create a generic IP whitelisting template'
layout: post
comments: true
author: Peter
date: 2018-12-10 11:59:07 +0000
slug: ''
wordpress_id: ''
tags: []
categories: []

---
At my current customer, we're working on securing the services in Azure. One of the requirements includes setting IP-filtering in different App Services.

This can be done in ARM by setting a property ipSecurityRestrictions property of the sites/config element for an App Service. You need to assign an array of items that include  the ipAddress (in CIDR format), action (Drop, Allow), priority (integer) and a name (optional, but useful for recognition of a rule in the networking blade of the Azure Portal).

The ranges are different for each service (accessible from Web Application Firewall, through internal proxy, other hostingprovider), but because the webapps that are deployed have a combination of different ranges we would like to administer the ranges in one central location.

Ideally, we can set a parameter for the LinkedTemplate that creates the App Service containing the ranges we want to whitelist.