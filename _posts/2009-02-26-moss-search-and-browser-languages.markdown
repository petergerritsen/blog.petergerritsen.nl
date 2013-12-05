---
author: admin
comments: true
date: 2009-02-26 21:03:01+00:00
layout: post
slug: moss-search-and-browser-languages
title: MOSS Search and browser languages
wordpress_id: 565
tags:
- MOSS
- Search
---

We had a very strange issue with some search functionality we developed for a portal. We created an option to search for documents in a library by specifying the path to a specific folder.

This is done by specifying a keyword query like _9:”folder/subfolder”_. In this case the searchproperty _9_ contains the path of the item.

While this worked well on our development environments and most of the browsers we tested on, a some browsers it failed. My colleague Arthur discovered it had something to do with the language setting of the browser. When we used Dutch (nl-NL) as preferred language the standard core search results web part didn’t return any items when using a query with a “/” in the conditions.

If you execute a KeyWordQuery or FullTextSqlQuery in your own code you have the possibility to specify a culture to use in the search. Unfortunately the core search results web part doesn’t provide you with an option to do this easily. In the end I came up with a “hack” to make this work. Just change the preferred languages the browser sends with each request in the OnInit of a webpart that inherits from the standards core results web part like so:

[sourcecode language="csharp"]
protected override void OnInit(EventArgs e)
{
	if (!string.IsNullOrEmpty(CultureOverrideTo))
	{
		if (this.Page.Request.UserLanguages.Length > 0)
			this.Page.Request.UserLanguages[0] = “en-US”;
	}

	base.OnInit(e);
}
[/sourcecode]

Note: Because this modifies the current request this can have some consequences on other web parts on your page. So use it with care.

UPDATE:

This same behavior can also occur in custom search code. To fix this, simply specify the Culture of your KeyWordQuery or FullTextSqlQuery.
