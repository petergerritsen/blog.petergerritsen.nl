---
author: Peter Gerritsen
comments: true
date: 2009-02-19 09:49:04+00:00
layout: post
slug: creating-a-3d-tagcloud-in-silverlight-part-2
title: Creating a 3D tagcloud in Silverlight (part 2)
wordpress_id: 566
tags:
- 3D
- JavaScript
- Silverlight
---

In [part 1](http://blog.petergerritsen.nl/2009/02/14/creating-a-3d-tagcloud-in-silverlight-part-1/) I showed you how to create the basics for a 3D tagcloud in Silverlight. In this part I’ll show how to get the tags from your html for inserting it into a blog template, we’ll change the color of the tag based on the weight of the tag and let the hyperlink button actually function as a hyperlink (until now it does nothing when you click on it).

There are a few different ways you can pass or get information from the page the Silverlight control is hosted in:

  * Set the initParams parameter in the Silverlight object definition
  * Read the HtmlDocument in your Silverlight code
  * Use JavaScript to call methods in your Silverlight code


You can only use the first option if your hosting your Silverlight application on the same domain. When using the hosting service on silverlight.live.com, no interaction between your application and HTML page is allowed.

For the second option you can use the HtmlPage object in your code to traverse through the HTML DOM, basic methods such as GetElementById are available.

The last option is the one I’ll be using in this post. To accomplish this you need to follow a few steps:

  * Decorate your page class with a "ScriptableType” attribute
  * Decorate the method you want to call with the “ScriptableMember” attribute and make sure it’s public
  * Register your object in the HtmlPage
  * Give your Silverlight object definition an id, so you can easily reference it from your JavaScript


The reason I’m going with this technique is the flexibility it provides when you want to reuse the tagcloud in other applications. Every blog engine has a different way to generate the HTML for a tagcloud, my blog just shows a list of tags.

To change the color based on the weight and set the url of the HyperlinkNutton we need to make some changes to the Tag3D object I showed you in the last post:

```csharp
public class Tag3D
{
    public HyperlinkButton btnLink { get; set; }
    private TextBlock textBlock { get; set; }
    public Point3D centerPoint { get; set; }
    private Color TagColor { get; set; }
    public Tag3D(double x, double y, double z, string text, string url, Color tagColor, int weight)
    {
        centerPoint = new Point3D(x, y, z);
        textBlock = new TextBlock();
        textBlock.Text = string.Format("{0} ({1})", text, weight);
        btnLink = new HyperlinkButton();
        btnLink.Content = textBlock;
        btnLink.NavigateUri = new Uri(url);
        this.TagColor = tagColor;
    }
    public void Redraw(double xOffset, double yOffset)
    {
        double zFactor = ((centerPoint.Z + 300) / 450.0);
        btnLink.FontSize = 30.0 * zFactor;
        double alpha = zFactor * 255;
        //Debug.WriteLine("Z: {0}; zFactor: {1}; alpha: {2}", centerPoint.Z, zFactor, alpha);
        btnLink.Foreground = new SolidColorBrush(Color.FromArgb(Convert.ToByte(alpha), TagColor.R, TagColor.G, TagColor.B));
        Canvas.SetLeft(btnLink, centerPoint.X + xOffset - (btnLink.ActualWidth / 2));
        Canvas.SetTop(btnLink, -centerPoint.Y + yOffset - (btnLink.ActualHeight/ 2));
        Canvas.SetZIndex(btnLink, Convert.ToInt32(centerPoint.Z));
    }
}
```

In the constructor we now pass in the url and a Color to show when the tag is the most important (I chose to use a scale of 1 to 10, with 10 being the most important).
The url is simply assigned to the NavigateUrl property of the HyperlinkButton. The Color is used when setting the new Foreground Brush. I also made some modifications in the calculations of the font size and alpha of the Brush to make it look a bit more realistic.
To let the JavaScript in the page add the tags I’ve created a AddTag method and decorated it with the ScriptableMember attribute:

```csharp
[ScriptableMember()]
public void AddTag(string tag, string url, int weight)
{
    if (weight > 10)
    weight = 10;
    Color color = new Color();
    color.R = Convert.ToByte(Math.Round(209.0 * ( weight / 10.0)));
    color.G = Convert.ToByte(Math.Round(18.0 * (weight / 10.0)));
    color.B = Convert.ToByte(Math.Round(65.0 * (weight / 10.0)));
    tagBlocks.Add(new Tag3D(0.0, 0.0, 0.0, tag, url, color));
}
```

In this method we calculate the Color of the tag based on the weight. Then we add a new tag to the tagBlocks list.

After calling this method a couple of times we need to place the tags and display them. I’ve changed the FillTags method shown in the previous post and renamed it to ProcessTags to make the name a bit more meaningful:

```csharp
[ScriptableMember()]
public void ProcessTags()
{
    double radius = RootCanvas.Width / 3;
    int max = tagBlocks.Count;
    double phi = 0;
    double theta = 0;
    for (int i = 1; i < max + 1; i++)
    {
        phi = Math.Acos(-1.0 + (2.0 * i – 1.0) / max);
        theta = Math.Sqrt(max * Math.PI) * phi;
        double x = radius * Math.Cos(theta) * Math.Sin(phi);
        double y = radius * Math.Sin(theta) * Math.Sin(phi);
        double z = radius * Math.Cos(phi);
        Tag3D tag = tagBlocks[i -1];
        tag.centerPoint = new Point3D(x, y, z);
        tag.Redraw(RootCanvas.Width / 2, RootCanvas.Height / 2);
        RootCanvas.Children.Add(tag.btnLink);
    }
}
```

We need one more thing to make the methods callable from JavaScript. Register the
object with the HtmlPage in the constructor:

```csharp
HtmlPage.RegisterScriptableObject("TagCloud", this);
```

No you can call the methods from JavaScript:

```javascript
function addTags() {
    var control = document.getElementById("Xaml1");
    control.content.TagCloud.AddTag("Silverlight", "http://silverlight.net", 5);
    control.content.TagCloud.AddTag("Tagcloud", "http://blogs.tamtam.nl", 2);
    control.content.TagCloud.AddTag("Tam Tam", "http://www.tamtam.nl", 10);
    control.content.TagCloud.AddTag("Axelerate3D", "http://www.codeplex.com", 8);
    control.content.TagCloud.AddTag("WPF", "http://www.microsoft.com", 1);
    control.content.TagCloud.AddTag("SharePoint", "http://www.microsoft.com", 4);
    control.content.TagCloud.ProcessTags();
}
```

I’m just attaching some code to the onclick of a button and hard-coding the tags. Normally you would handle the onload of the document (or better yet the $(document).ready in jQuery) and get your tags from the Html to pass them to the Silverlight object.

And that wraps it up for this tutorial.
