---
author: Peter Gerritsen
comments: true
date: 2010-03-02 16:17:52+00:00
layout: post
slug: new-sample-project-for-sp2010wordautomation-ui
title: 'New sample project for SP2010 Word Automation: UI'
wordpress_id: 782
tags:
- CodePlex
- SP2010
- Word Automation
---

I’ve just published a [second sample solution](http://sp2010wordautomation.codeplex.com/releases/view/41267) for the SP2010 Word Automation project on CodePlex. This solution will add a button to the Ribbon when browsing document libraries:

[![Ribbon button](/images/old/image_thumb10.png)](/images/old/image9.png)

When the button is clicked a modal Dialog is shown that will allow the user to specify the options used for conversion:

[![Modal Dialog](/images/old/image_thumb11.png)](/images/old/image10.png)

When the Ok button is clicked the selected files will be added to a conversion job and the job will be started.

The dialog is launched by some javascript that is specified in the CommandUIHandler section of the ribbon button definition.

```xml
<commandUIHandler
Command="SP2010WA_Convert_Button"
CommandAction="javascript:function convertDocument() {
Sys.loadScripts(['/_layouts/SP2010WordAutomation.UI/SP2010WordAutomation.UI.js'], function() {
SP2010WordAutomation.UI.ConvertDocument();
});
}
convertDocument();"
EnabledScript="javascript:function oneOrMoreEnable() {
var items = SP.ListOperation.Selection.getSelectedItems();
var ci = CountDictionary(items);
return (ci > 0);
}
oneOrMoreEnable();" />
```

I’ve decided to use the beta version of the ASP.Net 4.0 AJAX client library to load the required scriptfile when it is actually needed. While this is not completely necessary in this case, because the amount of script in there is quite little, it could provide a speedboost because the browser won’t load and interpret the script when the page loads.

The definition also contains some script to enable the button only when one or more files are selected.

The following lists the script that is loaded and called when the button is clicked:

```javascript
Type.registerNamespace("SP2010WordAutomation.UI");
SP2010WordAutomation.UI.ConvertDocument = function () {
var items = SP.ListOperation.Selection.getSelectedItems();
var selectedItems = '';
var k;
for (k in items) {
selectedItems += '|' + items[k].id;
}
var options = {
url: '/_layouts/SP2010WordAutomation.UI/ConvertDocument.aspx?items=' + selectedItems + '&source;=' + SP.ListOperation.Selection.getSelectedList(),
title: 'Convert Documents',
allowMaximize: false,
showClose: true,
width: 600,
height: 480,
dialogReturnValueCallback: SP2010WordAutomation.UI.ConvertCallback
};
SP.UI.ModalDialog.showModalDialog(options);
}
SP2010WordAutomation.UI.ConvertCallback = function(result, target) {
SP.UI.Notify.addNotification(target, false);
SP.UI.ModalDialog.RefreshPage(result);
}
```

First I use the _Type.registerNamespace_ method that is provided by the standard SharePoint scriptlibrary to make sure I don’t override other methods with the same names.

In the _ConvertDocument_ function we then launch a SharePoint dialog that will load an ApplicationPage which provides the user with the options they can choose. The _ConvertCallback_ function which is called when the dialog passes a result will add a notification message to the main screen.

To see how this mechanism can be used, please refer to [this post](http://blogs.msdn.com/vesku/archive/2010/02/25/how-to-sharepoint-2010-js-client-object-model-and-ui-advancements.aspx) by Vesa Juvonen
