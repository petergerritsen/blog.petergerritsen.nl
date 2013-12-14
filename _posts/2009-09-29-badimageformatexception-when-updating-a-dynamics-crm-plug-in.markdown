---
author: Peter Gerritsen
comments: true
date: 2009-09-29 16:57:24+00:00
layout: post
slug: badimageformatexception-when-updating-a-dynamics-crm-plug-in
title: BadImageFormatException when updating a Dynamics CRM Plug-in
wordpress_id: 543
tags:
- Dynamics CRM
---

Today I was working on a Dynamics CRM 4.0 Plug-in we’ve developed in the past and I needed to update the plug-in on our CRM server.

When I loaded the assembly in the Plug-in Registration tool I got the following exception:

```
Unhandled Exception: System.BadImageFormatException: Could not load file or assembly 'Microsoft.Crm.Sdk, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35' or one of its dependencies.
An attempt was made to load a program with an incorrect format.
  at System.Reflection.Assembly._GetExportedTypes()
  at PluginRegistrationTool.AssemblyReader.RetrievePluginsFromAssembly(String path)
  at PluginRegistrationTool.AssemblyReader.RetrievePluginsFromAssembly(String path)
  at PluginRegistrationTool.RegistrationHelper.RetrievePluginsFromAssembly(String pathToAssembly)
  at PluginRegistrationTool.PluginRegistrationForm.btnLoadAssembly_Click(Object sender, EventArgs e)
```

Appearently this was due to the fact I was working on a 64 bit environment. When you place the tool in the 64bit folder of the sdk\bin folder, the error doesn’t occur anymore and you’ll be able to succesfully update your plug-in.
