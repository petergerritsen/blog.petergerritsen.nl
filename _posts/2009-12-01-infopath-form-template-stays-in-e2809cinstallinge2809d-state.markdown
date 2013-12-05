---
author: admin
comments: true
date: 2009-12-01 10:47:57+00:00
layout: post
slug: infopath-form-template-stays-in-%e2%80%9cinstalling%e2%80%9d-state
title: InfoPath Form Template stays in “installing” state
wordpress_id: 533
tags:
- InfoPath
- MOSS
---

When I was testing to deploy a solution containing some form templates I got an error. Not very strange, because I was testing it.

The main downside though was one of the templates remained in the installing state. Apparently the easiest way to remove this template is by using some custom code, in this case I just used a console application within my dev box:

[sourcecode language="csharp"]

static void Main(string[] args)
{
FormsService fs = SPFarm.Local.Services.GetValue("");

foreach (FormTemplate ft in fs.FormTemplates)
{
if (ft.Name.Contains("Blackberry"))
{
ft.Delete();
}
}
}
[/sourcecode]
