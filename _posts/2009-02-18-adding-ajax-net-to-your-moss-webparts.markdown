---
author: Peter Gerritsen
comments: true
date: 2009-02-18 12:53:37+00:00
layout: post
slug: adding-ajax-net-to-your-moss-webparts
title: Adding AJAX.Net to your MOSS WebParts
wordpress_id: 567
tags:
- AJAX
- ASP.Net
- MOSS
---

I know there are loads of posts on this subject already, but in this one I’ll try to give some useful tips on this subject.

First, you need to add AJAX.Net entries to your web.config of the MOSS site. A very easy way to do this is by using the [Ajaxify](http://www.codeplex.com/ajaxifymoss) [stsadm extensions](http://www.codeplex.com/ajaxifymoss).

Simply add the included wsp to your solutions and deploy. After that you can easily add the entries by running the _stsadm –addajaxmoss_ command. Unfortunately you need to add one entry manually, but that is given as part of the output when you run the command.

Next you need to add a ScriptManager to the pages with the AJAX enabled webparts. You can off course modify the masterpage. I chose to create a base web part that checks if a ScriptManager has been added to the page and adds it if necessary:

```csharp
protected override void OnInit(EventArgs e)
{
    base.OnInit(e);
    //Register the ScriptManager
    ScriptManager scriptManager = ScriptManager.GetCurrent(this.Page);
    if (scriptManager == null)
    {
        scriptManager = new ScriptManager();
        scriptManager.ID = “ScriptManager1″;
        scriptManager.EnablePartialRendering = true;
        this.Page.Form.Controls.AddAt(0, scriptManager);
    }
}
```

Because MOSS needs a lot of JavaScript to function properly, this unfortunately requires another fix. You can read more about it [here](http://msdn.microsoft.com/en-us/library/bb861877.aspx). I added the fix to the base webpart with the following code:

```csharp
protected override void CreateChildControls()
{
    //Add fix according to http://msdn2.microsoft.com/en-us/library/bb861877.aspx
    EnsurePanelFix();
}
private void EnsurePanelFix()
{
    if (this.Page.Form!= null)
    {
        String fixupScript = @”
        _spBodyOnLoadFunctionNames.push(“”_initFormActionAjax”");
        function _initFormActionAjax()
        {
            if (_spEscapedFormAction == document.forms[0].action)
            {
                document.forms[0]._initialAction = document.forms[0].action;
            }
        }
        var RestoreToOriginalFormActionCore = RestoreToOriginalFormAction;
        RestoreToOriginalFormAction = function()
        {
            if (_spOriginalFormAction != null)
            {
                RestoreToOriginalFormActionCore();
                document.forms[0]._initialAction = document.forms[0].action;
            }
        }”;
        ScriptManager.RegisterStartupScript(this, typeof(BaseWebPart), “UpdatePanelFixup”, fixupScript, true);
    }
}
```

Because all the content you want to include in an UpdatePanel needs to be added to the ControlTemplateContainer.Controls collection, it’s not possible to write out any html you need between controls in the UpdatePanel by using the standard HtmlTextWriter.WriteLine method in the Render method. The easiest way is to add LiteralControls to the controlcollection of the UpdatePanel (with thanks to my collegue Wouter Lemaire for giving me the tip). To make this even easier, I’ve added the following extension method to my project:

```csharp
public static void AddLiteral(this UpdatePanel updatePanel, string html)
{
    Literal lit = new Literal();
    lit.Text = html + “\r\n”;
    updatePanel.ContentTemplateContainer.Controls.Add(lit);
}
```

You can call this method from the CreateChildControls method in your web part like so:

```csharp
updatePanel.AddLiteral(“<table><tr>”);
updatePanel.AddLiteral(“<td></td><td>Ma</td><td>Di</td><td>Wo</td><td>Do</td><td>Vr</td></tr><tr>”);
updatePanel.AddLiteral(“<td>Ochtend</td><td>”);
```

To indicate progress you need some form of visual feedback to the user. For this you need to set the ProgressTemplate of the UpdatePanel. For this you need to create a class that implements the ITemplate interface.

Then you implement the InstantiateIn method to create a template:

```csharp
public void ITemplate.InstantiateIn(Control container)
{
    Label lbl = new Label();
    lbl.Text = “Progress….”;
    container.Controls.Add(lbl);
}
```

Then add an object of this class to the ProgressTemplate property of the UpdatePanel:

```csharp
updateProgress.ProgressTemplate = new ProgressTemplate();
```
