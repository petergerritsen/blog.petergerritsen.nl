---
author: admin
comments: true
date: 2009-09-03 12:44:39+00:00
layout: post
slug: silverlight-multiple-animations-on-one-property-through-transforms
title: 'Silverlight: Multiple animations on one property through Transforms'
wordpress_id: 544
tags:
- Silverlight
---

When you create two or more animations that work on the same property of an object, Silverlight will only use the last of the defined animations.

By using transforms you’re able to achieve the same effect anyway. For instance, I’ve got a rectangle that slides up and down by using an animation that works on the Canvas.Top property:

[sourcecode language="xml"]






















[/sourcecode]

This will create an effect like this:


[![Install Microsoft Silverlight](/wp-content/uploads/sl4wp-ph.png)](http://go.microsoft.com/fwlink/?LinkID=149156)



If I want to apply an animation that jiggles the rectangle back and forth over the X- and Y-axis, I can’t use the Canvas.Top property anymore. So instead, we’ll add a Transform to the object and animate the properties of the Transform:

[sourcecode language="xml"]































[/sourcecode]

The result of this will look like the following:


[![Install Microsoft Silverlight](/wp-content/uploads/sl4wp-ph.png)](http://go.microsoft.com/fwlink/?LinkID=149156)
