---
author: Peter Gerritsen
comments: true
date: 2009-02-13 23:02:38+00:00
layout: post
slug: creating-a-3d-tagcloud-in-silverlight-part-1
title: Creating a 3D tagcloud in Silverlight (part 1)
wordpress_id: 568
tags:
- 3D
- Silverlight
---

When I saw the [wp-cumulus plugin by Roy Tanck](http://www.roytanck.com/2008/03/15/wp-cumulus-released/), I thought it would be a great idea to implement the same sort of functionality in Silverlight. It’s hardly original but allows me to learn some parts of the Silverlight framework.

The components behind it are quite simple:

  * Get (or send) the tags from your HTML page to the Silverlight usercontrol
  * Render the tags so it looks 3D
  * Create a method to rotate the tags based on the position of your mouse

#### Choosing a 3D library

The current version of Silverlight doesn’t include 3D functionality like WPF does through the [Media3D namespace](http://msdn.microsoft.com/en-us/library/system.windows.media.media3d.aspx). Fortunately some developers implemented the same functionality in libraries for Silverlight. The main options I found were Kit3D and Axelerate3D. I decided to use the last one because that one mimics the RotateTransform3D class in WPF 3D the best (it contains a TryTransform method).

#### Rendering the tags

I decided to tackle the second item first, because if I wasn’t able to manage this, the other items wouldn’t be very useful.

To create a tag in 3D you need some basic functionality:

  * A way to store it’s x, y and z coordinates
  * A hyperlinkbutton to redirect to a page that shows all the items with that tag
  * A textblock to display the tag

```csharp
public class Tag3D
{
public Tag3D(double x, double y, double z, string text)
{
centerPoint = new Point3D(x, y, z);
textBlock = new TextBlock();
textBlock.Text = text;
btnLink = new HyperlinkButton();
btnLink.Content = textBlock;
}

public HyperlinkButton btnLink { get; set; }

public TextBlock textBlock { get; set; }

public Point3D centerPoint { get; set; }

}
```

Then we need a way to make it look like it’s rendered in 3D. We do that by changing the fontsize and the opacity of the text. For that I created a method Redraw:

```csharp
public void Redraw(double xOffset, double yOffset)
{
double posZ = centerPoint.Z + 200;
btnLink.FontSize = 10 * (posZ / 100);
double alpha = centerPoint.Z + 200;
if (alpha > 255)
alpha = 255;

if (alpha < 0)
alpha = 0;

btnLink.Foreground = new SolidColorBrush(Color.FromArgb(Convert.ToByte(alpha), 0, 0, ));
Canvas.SetLeft(btnLink, centerPoint.X + xOffset – (btnLink.ActualWidth / 2));
Canvas.SetTop(btnLink, -centerPoint.Y + yOffset – (btnLink.ActualHeight/ 2));
Canvas.SetZIndex(btnLink, Convert.ToInt32(centerPoint.Z));
}
```

##### Placing the tags


To distribute the tags evenly over the sphere, we need some math. Luckily someone was way ahead of me and posted a useful [blogentry](http://blog.massivecube.com/?p=9.) on this subject (this technique is also used in the wp-cumulus plugin).

The following method creates and places the tags in the canvas:

```csharp
private void FillTags()
{
tagBlocks = new List();

string[] tags = new string[] { “Silverlight”,
“WPF”,
“3D”,
“Rotation”,
“SharePoint”,
“.Net”,
“C#”,
“Transform”,
“Blog”,
“TagCloud”,
“Tam Tam”,
“Axelerate3D”,
“MOSS”,
“Math”};

double radius = RootCanvas.Width / 3;
int max = tags.Length;
double phi = 0;
double theta = 0;

for (int i = 1; i < max + 1; i++)
{
phi = Math.Acos(-1.0 + (2.0 * i – 1.0) / max);
theta = Math.Sqrt(max * Math.PI) * phi;
double x = radius * Math.Cos(theta) * Math.Sin(phi);
double y = radius * Math.Sin(theta) * Math.Sin(phi);
double z = radius * Math.Cos(phi);

Tag3D tag = new Tag3D(x, y, z, tags[i -1]);
tag.Redraw(RootCanvas.Width / 2, RootCanvas.Height / 2);
RootCanvas.Children.Add(tag.btnLink);
tagBlocks.Add(tag);
}
}
```

At the moment the tags to render are hard-coded but we’ll sort that out in part 2.

**Rotating the tags**

To rotate the tags we will use the position of the mouse as a starting point. When the mousepointer is in the center the tagcloud will remain in the current position. Once the mouse is further away from the centerpoint we’ll increase the rotationspeed. The location of the mousepointer compared to the centerpoint will set the angle of the rotation.

First we will set the rotation when the tagcloud loads:

```csharp
void TagCloud_Loaded(object sender, RoutedEventArgs e)
{
FillTags();
rotateTransform = new RotateTransform3D();
rotateTransform.Rotation = new AxisAngleRotation3D(new Vector3D(1.0, 0.0, 0.0), 0);
CompositionTarget.Rendering += new EventHandler(CompositionTarget_Rendering);
LayoutRoot.MouseEnter += new MouseEventHandler(LayoutRoot_MouseEnter);
LayoutRoot.MouseLeave += new MouseEventHandler(LayoutRoot_MouseLeave);
}
```

Here we set the rotation angle to 0 and the rotationaxis to the x-axis. When the mouse moves, we’ll change those parameters, so the rotation will have an effect:

```csharp
void LayoutRoot_MouseMove(object sender, MouseEventArgs e)
{
Point mouseLocation = e.GetPosition(RootCanvas);
double relativeX = mouseLocation.X – (RootCanvas.ActualWidth / 2);
double relativeY = mouseLocation.Y – (RootCanvas.ActualHeight / 2);
MouseX.Text = relativeX.ToString();
MouseY.Text = relativeY.ToString();
double speed = Math.Sqrt(Math.Pow(relativeX, 2) + Math.Pow(relativeY, 2)) / 170;
RotationSpeed.Text = speed.ToString();
rotateTransform.Rotation = new AxisAngleRotation3D(new Vector3D(relativeY, relativeX, 0), speed);
}
```

To trigger the movement, we have to capture the MouseEnter and MouseLeave events:

[sourcecode language="c#"]
void LayoutRoot_MouseLeave(object sender, MouseEventArgs e)
{
     LayoutRoot.MouseMove -= LayoutRoot_MouseMove;
     runRotation = false;
}

void LayoutRoot_MouseEnter(object sender, MouseEventArgs e)
{
     LayoutRoot.MouseMove += new MouseEventHandler(LayoutRoot_MouseMove);
     runRotation = true;
}
```

Now that the rotationparameters are set we need to rotate the tags, or more precisely the centerpoint of the tag. To accomplish this we’ll make use of the Rendering event of the CompositionTarget object. This is called everytime the Silverlight plugin wants to render a new frame.

```csharp
void CompositionTarget_Rendering(object sender, EventArgs e)
{
    if (runRotation)
    {
        if (((AxisAngleRotation3D)rotateTransform.Rotation).Angle > 0.05)
        RotateBlocks();
    }
}

private void RotateBlocks()
{
foreach (Tag3D textBlock in tagBlocks)
{
Point3D newPoint;
if (rotateTransform.TryTransform(textBlock.centerPoint, out newPoint))
{
textBlock.centerPoint = newPoint;
textBlock.Redraw(RootCanvas.ActualWidth / 2, RootCanvas.ActualHeight / 2);
}
}
}
```

To relieve the CPU a bit, we’ll only rotate the tags if the rotation angle is higher than a threshold value. The actual transformation is accomplished by invoking the TryTransform method and passing it the current centerpoint of each tag.

At the moment the Silverlight control looks like this:













In the [next part](http://blog.petergerritsen.nl/2009/02/19/creating-a-3d-tagcloud-in-silverlight-part-2/) I’ll show you a way to dynamically set the tags, base their fontsize on the actual weight of the tag and actually use the hyperlink button.
