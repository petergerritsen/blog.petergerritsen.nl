---
author: Peter Gerritsen
comments: true
date: 2009-03-13 11:47:50+00:00
layout: post
slug: using-a-template-plugin-for-jquery-to-parse-json-data
title: Using a template plugin for jQuery to parse JSON data
wordpress_id: 564
tags:
- AJAX
- JavaScript
- jQuery
---

When you’re building an AJAX control in .Net there a a few possibilities. One of them is using AJAX.Net updatepanels. This saves you from writing tedious javascript code to refresh parts of you page. With the arrival of javascript libraries such as jQuery it’s much easier to create the AJAX functionality you want with javascript. However, you still have to write quite a lot of DOM manipulation code and use string concatenation
to process any JSON results and render the correct HTML.

Fortunately some javascript template engines are developed to make this easier. These engines come in all shapes and sizes, ranging from engines with an own templating syntax to simple data binding engines.

On the first side of the spectrum, there are engines such as [jTemplates](http://jtemplates.tpython.com/), this one uses python like syntax to create the instructions. On the other end engines like [Chain.js](http://wiki.github.com/raid-ox/chain.js) and [PURE](http://beebole.com/pure/) live, these can be considered more a databinding egines. The last ones make use of classnames for the databinding, like in the following Chain.js example:

```html
<div id="quickdemo">
	<div class="item">
		<span class="library">Library Name</span>
	</div>
</div>
```
```javascript
$('#quickdemo').items( [
		{library:'Prototype'},
		{library:'jQuery'},
		{library:'Dojo'},
		{library:'MooTools'}
	]).chain();
```

In this case the library field in the JSON objects is put into the element with “library” in de classname. The nice thing about Chain.js is the fact that it monitors the items collection for changes. Adding or removing items from script, automatically updates the generated HTML. So filtering and sorting can be very easily accomplished, some very easy to follow examples are available on the companion website.

PURE uses the same classnames based system for the databinding. Consider the following example:

```html
<ol class="siteList reference@id">
	<li class="sites">
		<a class="name url@href" href="http://beebole.com">Beebole</a>
	</li>
</ol>
```
```javascript
var data = {
	"reference": "3456",
	"sites": [{
			"name": "Beebole",
			"url": "http://beebole.com"
		},
		{
			"name": "BeeLit",
			"url": "http://beeLit.com"
		},
		{
			"name": "PURE",
			"url": http://beebole.com/pure
		}]
	};
$('ol.siteList').autoRender(data);
```

The _url@href_ and _reference@id_ classnames provide a way to set attributes of the databound elements.

But what if you don’t want to decorate your HTML elements with extra classnames to support the databinding? Or you want to handle some events of the generated elements?

For this PURE supports directives. You create your directives and pass them into the autoRender or render functions. Consider the following example:

```html
<div style="display: none;" id="bpvcategorytemplate">
	<ul>
		<li class="context">
			<a category="" class="context context@category" href="#">laden...</a>
		</li>
	</ul>
</div>
```
```javascript
function showProductCategories()
{
    $.getJSON(bpvweburl + "/_layouts/ecabointranet2009/bpv.ashx", {type: "categories"}, function(data)
    {
        var categoriescontainer = $("#bpvcategoriescontainer");
        categoriescontainer.empty();
        var list = categoriescontainer.append($("#bpvcategorytemplate").html());
        var directive = {'a.context[onclick]' : '"showProducts(this); return false;"'}
        list.autoRender( data, directive );
    });
}
```

Here we get some JSON from a handler, put the html from the template into a new element, and bind the JSON to that template. When binding the JSON data, we attach a javascript function to the onclick event of the generated anchor tags.

Much more can be accomplished by using directives, such as creating an alternating row style by setting a class during the binding of the items. For more information about using directives in PURE, read [this](http://wiki.github.com/pure/pure/what-is-a-directive) page.

There are loads more things possible in PURE or in the other engines, the best way to find out is to read the docs and try some things out.
