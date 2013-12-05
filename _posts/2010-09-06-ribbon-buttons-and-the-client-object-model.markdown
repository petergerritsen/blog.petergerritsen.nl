---
author: admin
comments: true
date: 2010-09-06 11:03:17+00:00
layout: post
slug: ribbon-buttons-and-the-client-object-model
title: Ribbon buttons and the Client Object Model
wordpress_id: 832
tags:
- Ribbon buttons
- Sandboxed
- SP2010
---

One of the standard Ribbon buttons in SharePoint 2010 allows you to mail a link to a document. Unfortunately the dev team at Microsoft didn’t update the functionality of this button to use one of the enhanced UI features: multiple selection. When you select 2 or more items, the button is automatically disabled.

To bypass this inconvenience, we decided to implement our own version of the button to replace the standard button. Replacing a button is actually quite easy with the UI extensibility options in SP2010.

First of, I started with an empty SharePoint project in Visual Studio 2010. I chose to create a sandboxed solution. Because we need to deploy and inject a JavaScript file on every page, I decided to go for the solution Jan Tielens introduced in his blog post ‘[Deploying and using jQuery with a SharePoint 2010 Sandboxed Solution](http://weblogs.asp.net/jan/archive/2010/09/02/deploying-and-using-jquery-with-a-sharepoint-2010-sandboxed-solution.aspx)’.

First of I added an empty element item to the project called Buttons. In this element I included the XML to hide the existing button and inject my own button:

[code]
<CustomAction
Id="TTSendLinksEmailRemoveRibbonButton"
Location="CommandUI.Ribbon">
<CommandUIExtension>
<CommandUIDefinitions>
<CommandUIDefinition
Location="Ribbon.Documents.Share.EmailItemLink" />
</CommandUIDefinitions>
</CommandUIExtension>
</CustomAction>
<CustomAction
Id="TTSendLinksEmailButton"
Location="CommandUI.Ribbon"
Sequence="15"
Title="E-mail a link">
<CommandUIExtension>
<CommandUIDefinitions>
<CommandUIDefinition Location="Ribbon.Documents.Share.Controls._children">
<Button
Id="Ribbon.Documents.Share.TTSendLinksEmailButton"
Alt="$Resources:core,cui_ButEmailLink;"
LabelText="$Resources:core,cui_ButEmailLink;"
ToolTipTitle="$Resources:core,cui_ButEmailLink;"
ToolTipDescription="$Resources:core,cui_STT_ButEmailLinkDocument;"
Sequence="15"
Command="TT_SendLinksEmail_Button"
Image16by16="/_layouts/$Resources:core,Language;/images/formatmap16x16.png"
Image16by16Top="-16"
Image16by16Left="-88"
Image32by32="/_layouts/$Resources:core,Language;/images/formatmap32x32.png"
Image32by32Top="-128"
Image32by32Left="-448"
TemplateAlias="o1" />
</CommandUIDefinition>
</CommandUIDefinitions>
<CommandUIHandlers>
<CommandUIHandler
Command="TT_SendLinksEmail_Button"
CommandAction="javascript:function sendLinksMail() {
TamTam_SP2010_SendLinksMail_SendLinksMail();
}
sendLinksMail();"
EnabledScript="javascript:function oneOrMoreEnable() {
var items = SP.ListOperation.Selection.getSelectedItems();
var ci = CountDictionary(items);
return (ci > 0);
}
oneOrMoreEnable();" />
</CommandUIHandlers>
</CommandUIExtension>
</CustomAction>
[/code]

For my own button, I copied the resourcestrings from the original XML that is in CMDUI.xml (14\TEMPLATE\GLOBAL\XML). This way the button gets the same look and feel and localized labels as the original button. When pressing the button the JavaScript function TamTam_SP2010_SendLinksMail_SendLinksMail is called. The EnabledScript enables the button when 1 or more items are selected.

To make sure the JavaScript is included, we’ll use a CustomAction element with ScriptLink as location (for more information see the previously mentioned blogpost by Jan Tielens):

[code]
<CustomAction
ScriptSrc="~SiteCollection/SiteAssets/TamTam.SP2010.EmailALinkMultiple.js"
Location="ScriptLink"
Sequence="10">
</CustomAction>
[/code]

Now all we have to do is create the necessary JavaScript. The out-of-the-box functionality opens the users email client with the link to the selected document in the body of the message.

To create the body of the message we’ll need to retrieve the link for every selected item by use of the Client Object Model. First we’ll get the ClientContext, the SPSite, the list were in and the id’s of the selected items:

[code]
TamTam_SP2010_SendLinksMail_SendLinksMail = function () {
this.selectedItems = SP.ListOperation.Selection.getSelectedItems();
this.selectedListGuid = SP.ListOperation.Selection.getSelectedList();

this.context = SP.ClientContext.get_current();

this.site = this.context.get_site();
this.context.load(this.site);

this.web = this.context.get_web();

this.selectedList = this.web.get_lists().getById(this.selectedListGuid);

}
[/code]

We then need to get the listitem object for every selected item. Because this is an asynchronous operation we’ll create an array, include every selected listitem in there and tell the context to load each item before calling executeQueryAsync:

[code]
this.selectedFiles = new Array();

var k;

for (k in this.selectedItems) {
this.selectedFiles.push(this.selectedList.getItemById(this.selectedItems[k].id).get_file());
this.context.load(this.selectedFiles[k]);
}

this.context.executeQueryAsync(Function.createDelegate(this, TamTam_SP2010_SendLinksMail_onQuerySucceeded), Function.createDelegate(this, TamTam_SP2010_SendLinksMail_onQueryFailed));
[/code]

The in the onQuerySucceeded callback function we can get the url’s for the items and construct the email:

[code]
TamTam_SP2010_SendLinksMail_onQuerySucceeded = function () {
var siteUrl = this.site.get_url();

var k;
var bodystring = "";
for (k in this.selectedFiles) {
var fileUrl = this.selectedFiles[k].get_serverRelativeUrl();

bodystring += siteUrl + fileUrl + "%0d%0a%0d%0a";
}

window.open('mailto:?body=' + bodystring);
}
[/code]
