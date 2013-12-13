---
author: Peter Gerritsen
comments: true
date: 2009-10-30 13:48:04+00:00
layout: post
slug: datetimefield-vs-fieldvalue-in-a-publishing-page
title: DateTimeField vs. FieldValue in a publishing page
wordpress_id: 542
tags:
- MOSS
categories: SharePoint
---

In one of our projects we needed to show the Modified date and time value in publishing page.

Not a big deal you would think. The only issue we had when using a FieldValue control is that the time that would be displayed was 1 or 2 hours later than the actual value.

After I changed this to a DateTimeField the difference was gone.

The only thing you need to add is a ControlMode=”Display” attribute to prevent editors from setting their own value in edit mode.

The difference is in the way the value is rendered. The FieldValue control does a ToString() and HtmlEncode on the value. The DateTimeField actually converts the value to local time by using the TimeZone class.

It’s all in the details!
