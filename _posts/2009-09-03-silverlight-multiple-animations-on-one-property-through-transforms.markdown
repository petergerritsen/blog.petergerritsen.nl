---
author: Peter Gerritsen
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

```xml
<userControl x:Class="TestAnimationTransform.MainPage"
xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
xmlns:d="http://schemas.microsoft.com/expression/blend/2008" xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
mc:Ignorable="d" d:DesignWidth="640" d:DesignHeight="480">
<userControl.Resources>
<storyboard x:Name="WaveTop" AutoReverse="True" RepeatBehavior="Forever">
<doubleAnimationUsingKeyFrames x:Name="WaveAnimationTop" BeginTime="00:00:00" Storyboard.TargetName="Rectangle" Storyboard.TargetProperty="(canvas.top)">
<easingDoubleKeyFrame KeyTime="00:00:00" Value="50">
<easingDoubleKeyFrame.EasingFunction>
<sineEase EasingMode="EaseInOut"/>
</easingDoubleKeyFrame.EasingFunction>
</easingDoubleKeyFrame>
<easingDoubleKeyFrame KeyTime="00:00:10" Value="400">
<easingDoubleKeyFrame.EasingFunction>
<sineEase EasingMode="EaseInOut"/>
</easingDoubleKeyFrame.EasingFunction>
</easingDoubleKeyFrame>
</doubleAnimationUsingKeyFrames>
</storyboard>
</userControl.Resources>
<canvas x:Name="LayoutRoot">
<rectangle x:Name="Rectangle" Fill="Blue" Width="50" Height="50" Canvas.Top="240" Canvas.Left="200">
</rectangle>
</canvas>
<userControl>
```

This will create an effect like this:


[![Install Microsoft Silverlight](/wp-content/uploads/sl4wp-ph.png)](http://go.microsoft.com/fwlink/?LinkID=149156)



If I want to apply an animation that jiggles the rectangle back and forth over the X- and Y-axis, I can’t use the Canvas.Top property anymore. So instead, we’ll add a Transform to the object and animate the properties of the Transform:

```xml
<storyboard x:Name="WaveJiggle" AutoReverse="True" RepeatBehavior="Forever">
<doubleAnimationUsingKeyFrames x:Name="XAnimation" BeginTime="00:00:00" Storyboard.TargetName="TranslateTransform" Storyboard.TargetProperty="X">
<easingDoubleKeyFrame KeyTime="00:00:00" Value="20">
<easingDoubleKeyFrame.EasingFunction>
<sineEase EasingMode="EaseInOut"/>
</easingDoubleKeyFrame.EasingFunction>
</easingDoubleKeyFrame>
<easingDoubleKeyFrame KeyTime="00:00:01" Value="80">
<easingDoubleKeyFrame.EasingFunction>
<sineEase EasingMode="EaseInOut"/>
</easingDoubleKeyFrame.EasingFunction>
</easingDoubleKeyFrame>
</doubleAnimationUsingKeyFrames>
<doubleAnimationUsingKeyFrames x:Name="YAnimation" BeginTime="00:00:00" Storyboard.TargetName="TranslateTransform" Storyboard.TargetProperty="Y">
<easingDoubleKeyFrame KeyTime="00:00:00" Value="20">
<easingDoubleKeyFrame.EasingFunction>
<sineEase EasingMode="EaseInOut"/>
</easingDoubleKeyFrame.EasingFunction>
</easingDoubleKeyFrame>
<easingDoubleKeyFrame KeyTime="00:00:01" Value="80">
<easingDoubleKeyFrame.EasingFunction>
<sineEase EasingMode="EaseInOut"/>
</easingDoubleKeyFrame.EasingFunction>
</easingDoubleKeyFrame>
</doubleAnimationUsingKeyFrames>
</storyboard>
<rectangle x:Name="Rectangle" Fill="Blue" Width="50" Height="50" Canvas.Top="240" Canvas.Left="200">
<rectangle.RenderTransform>
<translateTransform X="50" Y="50" x:Name="TranslateTransform"/>
</rectangle.RenderTransform>
</rectangle>
```

The result of this will look like the following:


[![Install Microsoft Silverlight](/wp-content/uploads/sl4wp-ph.png)](http://go.microsoft.com/fwlink/?LinkID=149156)
