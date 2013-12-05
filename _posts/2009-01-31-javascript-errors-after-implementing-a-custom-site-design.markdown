---
author: Peter Gerritsen
comments: true
date: 2009-01-31 11:44:11+00:00
layout: post
slug: javascript-errors-after-implementing-a-custom-site-design
title: JavaScript errors after implementing a custom site design
wordpress_id: 572
tags:
- CSS
- JavaScript
- MOSS
---

Most of the times we create and develop a custom design for the portals and websites we build.

This can actually lead to some rather odd JavaScript error messages saying some object is undefined when dragging and dropping web parts or using the list item edit menu.

Believe it or not, this is most of the times due to the custom style sheet and not to any custom JavaScript.

The reason for this is that the build in JavaScript of MOSS use a property offsetParent of a DOM-element to perform some "magic". **offsetParent** returns a reference to the object which is the closest (nearest in the containment hierarchy) positioned containing element. In some of our custom designs this sometimes returns null and the script throws the "object undefined" error.

The easiest solution to prevent this, is to absolute-position the body by include the following statement in your css:

```css
body { position: absolute !important; } 
```

This can off course lead to some css issues, so positioning a wrapper is also an option.
