---
author: Peter Gerritsen
comments: true
date: 2009-03-24 10:01:32+00:00
layout: post
slug: jquery-performance-optimization-tips
title: jQuery performance optimization tips
wordpress_id: 563
tags:
- jQuery
categories: Javascript
---

Sometimes all the cool effects and functionality you’ve build on top of jQuery turn out to be a bit sluggish. Here are a few tips to optimize performance.

1) Use jQuery 1.3+. This one utilizes the new Sizzle selector engine which is much faster then the selector engine in the jQuery versions before that.

2) Make your selectors as specific as possible. Sizzle is build to fail as soon as possible. Using id’s as part of your selector statement can provide quick performance gains. Also specifying a scope for your selector can be beneficial as jQuery only iterates over the elements within the scope.

3) Reuse selectors and use chaining. Storing the result of a selector in an object variable makes reuse easy. Chaining allows you to perform multiple actions on the already selected DOM elements.

4) Don’t use class-only selectors. $(“.active”) iterates over all DOM elements. So use a tag name to narrow the search down, $(“li.active”).

5) Use the [profiler plugin](http://ejohn.org/blog/deep-profiling-jquery-apps/) by John Resig to profile your jQuery calls.
