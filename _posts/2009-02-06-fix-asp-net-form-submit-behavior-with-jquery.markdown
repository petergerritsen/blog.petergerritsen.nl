---
author: Peter Gerritsen
comments: true
date: 2009-02-06 14:26:49+00:00
layout: post
slug: fix-asp-net-form-submit-behavior-with-jquery
title: Fix ASP.Net form submit behavior with jQuery
wordpress_id: 570
tags:
- ASP.Net
- jQuery

---

In standard ASP.Net web form pages there's only one form tag for the entire page. This unfortunately has some side effects on form submit behavior.

In a standard HTML all forms are contained in their own form tag. When the browser receives a enter key press for the form the form is submitted. Because of the single form in a ASP.Net web form this behavior will be broken and the first submit button on the page will always be triggered by the browser when pressing enter.

Microsoft has included a feature in ASP.Net 2.0 to overcome this problem. In short you add an attribute _DefaultButton_, with the ID of the button to trigger, to an ASP Panel control that wraps the form. Unfortunately this solution doesn't work in Firefox.

To fix this, I decided to use jQuery. What we need to have is a way to identify the different forms on a page and connect them with the right submit button. So I added a fieldset tag around the single form:

```csharp
writer.WriteLine("<div class=\"regular_forms\">");

writer.WriteLine("", btnSearch.ClientID);

// form contents go here
```

The control _btnSearch_ is the one we want to trigger when a user presses the enter button.

To hook up the button to the form we use the following JavaScript/jQuery:



This script finds all fieldset elements containing the _defaultsubmitbutton_ attribute, locates all textboxes and password fields within that fieldset and hooks up the keydown event.
When the enter key is pressed (keycode 13) the default event is canceled and depending on the type of button the right postback method is triggered.
