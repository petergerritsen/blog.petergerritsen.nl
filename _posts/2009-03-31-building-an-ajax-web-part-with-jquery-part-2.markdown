---
author: Peter Gerritsen
comments: true
date: 2009-03-31 10:29:31+00:00
layout: post
slug: building-an-ajax-web-part-with-jquery-part-2
title: Building an AJAX web part with jQuery (Part 2)
wordpress_id: 559
tags:
- AJAX
- ASP.Net
- jQuery
- MOSS
---

In [part 1](http://blog.petergerritsen.nl/2009/03/30/building-an-ajax-web-part-with-jquery-part-1/) of this series I explained a bit about the context and goal of creating an AJAX web part without using ASP.Net AJAX. I also showed the steps necessary for creating services that return data in the JSON format. In this post I’ll show you how to call these services from JavaScript and insert the data in the HTML placeholders rendered by the web part.

**Calling the generic handler for products and categories**

jQuery offers various methods to perform asynchronous calls to web resources. To retrieve JSON the most used are jQuery.ajax and jQuery.getJSON. The last one uses a HTTP GET request and is simpler to use, the jQuery.ajax method offers more options/flexibility. For retrieving the product categories and the products, I’ve decided to go with getJSON.
The code for retreiving the categories looks as follows:

```javascript
function showProductCategories() {
$.getJSON(bpvweburl + "/_layouts/intranet2009/bpv.ashx",
{type: "categories"},
function(data)
{
var categoriescontainer = $("#bpvcategoriescontainer");
categoriescontainer.empty();
var list = categoriescontainer.append($("#bpvcategorytemplate").html());
var directive = {'a.context[onclick]' : '"showProducts(this);return false;"'};
list.autoRender( data, directive );
});
}
```

The first line is responsible for calling the handler. It specifies a inline callback method to handle the returned data. The JSON returned is processed by the PURE templating plugin. I’ve blogged about using this [before](http://blog.petergerritsen.nl/2009/03/13/using-a-template-plugin-for-jquery-to-parse-json-data/).

**Calling the shopping cart web service**

For getting the data from the web service, we’ll use the jQuery.ajax method. That’s because we need to do a HTTP POST request as well as specify some other options for the request. For more information see [this](http://encosia.com/2008/03/27/using-jquery-to-consume-aspnet-json-web-services/) post by Dave Ward. To initially load the shopping cart we use the following code:

```javascript
function loadShoppingcart()
{
$.ajax({
type: "POST",
url: "/_layouts/ecabointranet2009/bpvshoppingcart.asmx/GetItems",
data: "{}",
contentType: "application/json; charset=utf-8",
dataType: "json",
success: rendershoppingcart,
error: showError
});
}

function rendershoppingcart(msg) {
var cartcontainer = $("#bpvcartcontent");
cartcontainer.empty();

if (msg.d.length > 0)
{
var cartitemslist = cartcontainer.append($("#bpvcarttemplate").html());

var directives = {
'a.bpvedit[onclick]' : '"editAmount(this); return false;"',
'a.bpvremove[onclick]' : '"removeProduct(this); return false;"'
}
cartitemslist.autoRender( msg.d, directives );
}
else
{
cartcontainer.append("U heeft nog geen producten in uw winkelwagen");

}

$("#bpvcartcontent").effect("highlight", {color: "#ffcf57"}, 700, null);
}

function showError(xhr, status, error)
{
var err = eval("(" + xhr.responseText + ")");

$("#bpverrordialog span.errormessage").html(err.Message);

$("#bpverrordialog").dialog("open");
}
```

As you can see we post to the GetItems method of the asmx. We need to specify a empty JSON object as data (more information on that in [this](http://encosia.com/2008/06/05/3-mistakes-to-avoid-when-using-jquery-with-aspnet-ajax/) post, again by Dave Ward) and specify the contentType we want returned. When the AJAX call returns an error, we’ll call the showError method (which uses the jQuery UI dialog widget I’ll tell more about in part 3). When the call is successful, the rendershoppingcart method is called. This checks if the cart is empty, so we can display a message in that case, or uses PURE again for rendering the cart contents. To provide visual feedback to the user when the cart is updated, we’ll use the highlight effect on the cart to attract the attention of the user.
If we want to pass in parameters with an AJAX call (like the productId when we want to delete an item from the cart), we need to construct a parameters object and serialize that as string for usage in the call:

```javascript
var paramsdata = { "productId" : $("#bpvremoveitemid").val() }

$.ajax({
type: "POST",
url: "/_layouts/ecabointranet2009/bpvshoppingcart.asmx/DeleteItem",
data: JSON.stringify(paramsdata),
contentType: "application/json; charset=utf-8",
dataType: "json",
success: rendershoppingcart,
error: showError
});
```

As you can see, we use a JSON.stringify method for serializing the object as a string.

You can [download](http://www.json.org/json2.js) the scriptfile needed for this from JSON.org.

All the other methods in the web service are called in the same way, so there’s no need to inundate you with more code on that. The only thing left is to show you parts of the contents of the Render method in the webpart:

```csharp
// Templates
// Categorylist template
writer.WriteLine("

");
writer.WriteLine("

### Productcategorie

");
writer.WriteLine("

  * [laden...](\"#\")
");
writer.WriteLine("

");
// Productlist template
writer.WriteLine("

");
writer.WriteLine("

### Product

");
writer.WriteLine("

  * [geen items](\"#\")
");
writer.WriteLine("

");
// Shoppingcart template
writer.WriteLine("

");
writer.WriteLine("

### Informatie en bestellen

");
writer.WriteLine("

Product
Aantal
");
writer.WriteLine("

naam
aantal
[![](\"/_layouts/images/ecabo/2009/page-edit.gif\")](\"#\") [![](\"/_layouts/images/ecabo/2009/bin.gif\")](\"#\")
");
writer.WriteLine("");
writer.WriteLine("

");

// Item selector
writer.WriteLine("

");
writer.WriteLine("

## 1. Selecteer uw producten

");
writer.WriteLine("

");
writer.WriteLine("

### Productcategorie

");
writer.WriteLine("

### Product

");
writer.WriteLine("

");
writer.WriteLine("

### Informatie en bestellen

");
writer.WriteLine("![](\"/_layouts/images/blank.gif\")");
writer.WriteLine("");
writer.WriteLine("");
writer.WriteLine("Aantal:");
writer.WriteLine("");
writer.WriteLine("");
writer.WriteLine("");
writer.WriteLine("

");
writer.WriteLine("

");

// Shoppingcart
writer.WriteLine("

");
writer.WriteLine("

## 2. Lijst met uw bestelling

");
writer.WriteLine("

U heeft nog geen producten in uw winkelwagen

");

writer.WriteLine("[Alles verwijderen](\"#\")");
writer.WriteLine("

");
```

As you can see, all we do is write out HTML. First I write out the HTML needed for the databinding of the categories and products (I removed that because it’s the same as the category one). Then some placeholders and form elements are rendered. There’s a little more HTML off course, but you get the point, NO CODE :-)

In the [next part](http://blog.petergerritsen.nl/2009/03/31/building-an-ajax-web-part-with-jquery-part-3/) I’ll show you how to use some jQuery plugins to enhance the experience of the user and validation of the input.
