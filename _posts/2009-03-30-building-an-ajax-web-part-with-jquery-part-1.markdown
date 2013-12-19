---
author: Peter Gerritsen
comments: true
date: 2009-03-30 10:08:51+00:00
layout: post
slug: building-an-ajax-web-part-with-jquery-part-1
title: Building an AJAX web part with jQuery (Part 1)
wordpress_id: 562
tags:
- AJAX
- ASP.Net
- jQuery
- MOSS
categories: [SharePoint, Javascript]
---

In this series of posts I will describe how I build a web part full of AJAX functionality using jQuery and some plugins for jQuery.


###Why?


ASP.Net AJAX is quite hard to implement using only code. Besides that, having multiple updatepanels and multiple triggers outside of these updatepanels, can complicate stuff very quickly. So I decided to see how much time I would need to build the required functionality using JavaScript with jQuery.

###What?

The web part of choice was one that would allow visitors of the portal to order a bunch of products from different categories. Products are stored in a list and have a choice sitecolumn to specify the category of the item. Products can be selected and added to a shopping cart. After the wanted items are added to the shopping cart, the user can specify some comments, a handling method (Pick-up or deliver) and a delivery-/pickup date and send in the order. The order information is saved in a list, the user gets a confirmation email and the shopping cart is cleared. The resulting web part look like this (sorry for the dutch interface):

![Example interface](/images/old/snipping24.png)

###How?

For getting the categories and products we’ll use a generic handler. This offers the possibility of caching and is perfectly suitable for read-only data access. The shopping cart and order functionality will be provided through a ASP.Net Webservice. Both the handler and web service will return JSON data. We’ll need ASP.Net 3.5 for this.

The web part will only override the render method and write out the needed HTML. So no control instantiation, event handling or other code.

###Useful links

Dave Ward’s weblog on [encosia.com](http://encosia.com/) provides a vital source for information on combining jQuery and ASP.Net. Also [Rick Strahl](http://www.west-wind.com/Weblog/) has some useful posts on this subject.

**Building the products handler**

We’ll start by building the Generic Handler that returns JSON data for categories and products. We’ll differentiate between the two by passing in the type with the query string. I’ve created two classes for data retrieval from the list in SharePoint, one that returns the choices of the categories and one that returns the products based on the category. We’ll then use the DataContractJsonSerializer to generate and return the JSON for this data:

```csharp
public void ProcessRequest(HttpContext context)
{
	try
	{
		if (string.IsNullOrEmpty(context.Request["type"]))
			throw new ArgumentException("type not specified or null");
		
		string type = context.Request["type"];
		context.Response.Cache.SetExpires(DateTime.Now.AddSeconds(300));
		context.Response.Cache.SetCacheability(HttpCacheability.Public);
		
		if (type.Equals("categories", StringComparison.InvariantCultureIgnoreCase))
		{
			StringCollection categories = BPVProductCategory.GetProductCategories();
			DataContractJsonSerializer ser = new DataContractJsonSerializer(typeof(StringCollection));
			ser.WriteObject(context.Response.OutputStream, categories);
		}
		
		if (type.Equals("products", StringComparison.InvariantCultureIgnoreCase))
		{
			if (string.IsNullOrEmpty(context.Request["category"]))
				throw new ArgumentException("category not specified or null");
			
			string category = HttpUtility.UrlDecode(context.Request["category"]);
			Ecabo.Intranet2009.SharePoint.Diagnostics.Logging.LogMessageFormat("Category: {0}", category);
			List products = BPVProduct.GetAvailableProducts(category);
			
			DataContractJsonSerializer ser = new DataContractJsonSerializer(typeof(List));
			ser.WriteObject(context.Response.OutputStream, products);
		}
	}
	catch (Exception ex)
	{
		context.Response.Write(string.Format("ERROR: {0}", ex.Message));
	}
}
```


### Building the shopping cart web service


Next, we need to create a web service that will allows us to modify the shopping cart with add, remove, change and clear methods. This web service will also contain a method to place the order. To make sure this web service will return JSON when requested, we’ll decorate the class with the ScriptService attribute (normally you just have to comment out the automatically included line in the class definition) and we decorate
the methods with the ScriptMethod attribute in which we specify the ResponseFormat to be JSON. To store the shoppingcart between requests to the service we’ll use the Session. In order for that to work we add the EnableSession=true parameter to the WebMethod attribute of each method. The resulting code looks like this:

```csharp
[WebService(Namespace = "http://sharepoint.ecabo.nl/200903/")]
[WebServiceBinding(ConformsTo = WsiProfiles.BasicProfile1_1)]
[System.ComponentModel.ToolboxItem(false)]
// To allow this Web Service to be called from script, using ASP.NET AJAX, uncomment the following line.
[System.Web.Script.Services.ScriptService]
public class BPVShoppingCart : System.Web.Services.WebService
{
	private List ShoppingCart
	{
		get
		{
			if (HttpContext.Current.Session["BPVShoppingCart"] != null)
			{
				return (List)HttpContext.Current.Session["BPVShoppingCart"];
			}
			else
			{
				List shoppingCart = new List();
				HttpContext.Current.Session.Add("BPVShoppingCart", shoppingCart);
				return shoppingCart;
			}
		}
		set
		{
			HttpContext.Current.Session["BPVShoppingCart"] = value;
		}
	}

	[WebMethod(EnableSession = true)]
	[ScriptMethod(ResponseFormat = ResponseFormat.Json)]
	public List GetItems()
	{
		try
		{
			return ShoppingCart;
		}
		catch (Exception ex)
		{
			Ecabo.Intranet2009.SharePoint.Diagnostics.Logging.LogException(ex);
			return null;
		}
	}
	
	[WebMethod(EnableSession = true)]
	[ScriptMethod(ResponseFormat = ResponseFormat.Json)]
	public List AddItem(string productName, int productId, string productCode, int amount)
	{
		try
		{
			ShoppingCartItem item = ShoppingCart.Find(p => p.ProductID == productId);
			if (item == null)
			{
				item = new ShoppingCartItem();
				item.Amount = amount;
				item.ProductCode = productCode;
				item.ProductID = productId;
				item.ProductName = productName;
				ShoppingCart.Add(item);
			}
			else
			{
				item.Amount += amount;
			}
			return ShoppingCart;
		}
		catch (Exception ex)
		{
			Ecabo.Intranet2009.SharePoint.Diagnostics.Logging.LogException(ex);
			return null;
		}
	}
	
	[WebMethod(EnableSession = true)]
	[ScriptMethod(ResponseFormat = ResponseFormat.Json)]
	public List DeleteItem(int productId)
	{
		try
		{
			ShoppingCartItem item = ShoppingCart.Find(p => p.ProductID == productId);
			if (item != null)
			{
				ShoppingCart.Remove(item);
			}
			return ShoppingCart;
		}
		catch (Exception ex)
		{
			Ecabo.Intranet2009.SharePoint.Diagnostics.Logging.LogException(ex);
			return null;
		}
	}
	
	[WebMethod(EnableSession = true)]
	[ScriptMethod(ResponseFormat = ResponseFormat.Json)]
	public List ChangeAmount(int productId, int amount)
	{
		try
		{
			if (amount == 0)
			return DeleteItem(productId);
			ShoppingCartItem item = ShoppingCart.Find(p => p.ProductID == productId);
			if (item != null)
			{
				item.Amount = amount;
			}
			return ShoppingCart;
		}
		catch (Exception ex)
		{
			Ecabo.Intranet2009.SharePoint.Diagnostics.Logging.LogException(ex);
			return null;
		}
	}
	
	[WebMethod(EnableSession = true)]
	[ScriptMethod(ResponseFormat = ResponseFormat.Json)]
	public List ClearCart()
	{
		try
		{
			ShoppingCart.Clear();
			return ShoppingCart;
		}
		catch (Exception ex)
		{
			Ecabo.Intranet2009.SharePoint.Diagnostics.Logging.LogException(ex);
			return null;
		}
	}
	
	[WebMethod(EnableSession = true)]
	[ScriptMethod(ResponseFormat = ResponseFormat.Json)]
	public List PlaceOrder(string comments, string deliveryType, string deliveryDate)
	{
		try
		{
			string products = "";
			foreach (ShoppingCartItem item in ShoppingCart)
			{
				products += string.Format("{0} - {1} - {2}\r\n", item.ProductCode, item.Amount, item.ProductName);
			}
			
			BPVBestelling bestelling = new BPVBestelling();
			bestelling.Comments = comments;
			bestelling.Handling = deliveryType;
			bestelling.HandlingDate = Convert.ToDateTime(deliveryDate, new CultureInfo("nl-NL"));
			bestelling.Title = DateTime.Now.ToString("yyyyMMdd_HHmm") + "_" + SPContext.Current.Web.CurrentUser.Email;
			bestelling.Products = products;
			bestelling.PlacedBy = SPContext.Current.Web.CurrentUser;
			if (BPVBestelling.AddBestelling(bestelling))
			{
				BPVBestelling.SendConfirmationEmail(bestelling, ShoppingCart);
				ShoppingCart.Clear();
				return ShoppingCart;
			}
			else
			throw new Exception("Het is niet mogelijk om uw bestelling te verwerken");
		}
		catch (Exception ex)
		{
			Ecabo.Intranet2009.SharePoint.Diagnostics.Logging.LogException(ex);
			throw new Exception("Het is niet mogelijk om uw bestelling te verwerken", ex);
		}
	}
}
```

There’s one more thing needed for letting the web service return the data in JSON format. We need to include a httpHandler in the web.config for the asmx extension that routes the request to the ScriptHandlerFactory. An easy way to do this is for your SharePoint webapp by creating a blank web.config and placing it in a subfolder of the layouts directory where you also place the asmx file. The following is all you need in that web.config file:

```xml
<?xml version="1.0" standalone="yes"?>
<configuration>
	<system.web>
	<httpHandlers>
		<add verb="*" path="*.asmx" validate="false" type="System.Web.Script.Services.ScriptHandlerFactory, System.Web.Extensions, Version=3.5.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35"/>
	</httpHandlers>
	<compilation debug="true"/></system.web>
</configuration>
```

###Next parts

That wraps it up for this first post in the series. In [part 2](http://blog.petergerritsen.nl/2009/03/31/building-an-ajax-web-part-with-jquery-part-2/) I’ll show you how to call these services from jQuery and insert the data in the HTML of the webpart. In [part 3](http://blog.petergerritsen.nl/2009/05/16/what-asp-net-developers-should-know-about-jquery/) we’ll enhance the experience by including dialogs and validation in the solution.
