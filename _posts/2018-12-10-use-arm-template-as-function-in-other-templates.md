---
title: 'ARM: Create a generic IP whitelisting template'
layout: post
comments: true
author: Peter
date: 2018-12-10 11:59:07 +0000
slug: ''
wordpress_id: ''
tags: []
categories: []

---
At my current customer, we're working on securing the services in Azure. One of the requirements includes setting IP-filtering in different App Services.

This can be done in ARM by setting a property ipSecurityRestrictions property of the sites/config element for an App Service. You need to assign an array of items that include  the ipAddress (in CIDR format), action (Drop, Allow), priority (integer) and a name (optional, but useful for recognition of a rule in the networking blade of the Azure Portal).

The ranges are different for each service (accessible from Web Application Firewall, through internal proxy, other hostingprovider), but because the webapps that are deployed have a combination of different ranges we would like to administer the ranges in one central location.

Ideally, we can set a parameter for the LinkedTemplate that creates the App Service containing the ranges we want to whitelist.

We solved that in the following way:

Create a template with a variable per range you want to support (ip's are chosen random):

```json
"variables": {
    "EmptyRange": [],
    "LocalOutgoingRange": [
      {
        "ipAddress": "133.144.155.0/23",
        "action": "Allow",
        "name": "SGRange"
      }
    ],
    "WAFRange": [
      {
        "ipAddress": "99.100.101.0/21",
        "action": "Allow",
        "name": "WAFRange"
      },
      {
        "ipAddress": "100.100.100.0/19",
        "action": "Allow",
        "name": "WAFRange"
      }
    ]
```

Add a parameter to the template that contains the name of one or more ranges the deployment needs to add:

```json
 "parameters": {
    "ipWhitelistingSets": {
      "type": "string",
      "defaultValue": "",
      "metadata": {
        "description": "Comma separated list of values LocalOutgoingRange,WAFRange"
      }
    },
```

Next, create two variables. the first one splits the parameter ipWhitelistingSets into an array. 
The next one uses this to conditionally concat the values of the ranges. If the range is not specified, the empty array is concatted, basically a NOOP.

```json
 	"RangesToUse": "[split(parameters('ipWhitelistingSets'), ',')]",
    "ConcattedRange": "[concat(if(contains(variables('RangesToUse'), 'LocalOutgoingRange'), variables('LocalOutgoingRange'), variables('EmptyRange')), if(contains(variables('RangesToUse'), 'WAFRange'), variables('WAFRange'), variables('EmptyRange'))]",
```

We can then use this variable to loop through and set the rules in the webapp:

```json
"resources": [
    {
      "comments": "IP whitelisting",
      "apiVersion": "2018-02-01",
      "location": "[resourceGroup().location]",
      "condition": "[not(empty(variables('RangesToUse')))]",
      "type": "Microsoft.Web/sites/config",
      "name": "[concat(parameters('webAppName'), '/web')]",
      "properties": {
        "mode": "Incremental",
        "copy": [
          {
            "name": "ipSecurityRestrictions",
            "count": "[length(variables('ConcattedRange'))]",
            "input": {
              "ipAddress": "[variables('ConcattedRange')[copyIndex('ipSecurityRestrictions')].ipAddress]",
              "action": "[variables('ConcattedRange')[copyIndex('ipSecurityRestrictions')].action]",
              "priority": "[copyIndex('ipSecurityRestrictions', 1)]",
              "name": "[variables('ConcattedRange')[copyIndex('ipSecurityRestrictions')].name]"
            }
          }
        ]
      }
    },
```

After this, we can call our template from the template that creates the App Service to set the ip restrictions:

```json
{
      "comments": "Whitelisting WebApp Elements",
      "apiVersion": "2015-01-01",
      "name": "[concat(concat(parameters('webAppName'), 'wl'))]",
      "type": "Microsoft.Resources/deployments",
      "condition": "[not(empty(parameters('ipWhitelistingSets')))]",
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[variables('ipWhitelistingTemplateUrl')]",
          "contentVersion": "1.0.0.0"
        },
        "parameters": {
          "webAppName": {
            "value": "[parameters('webAppName')]"
          },
          "ipWhitelistingSets": {
            "value": "LocalOutgoingRange,WAFRange"
          }          
        }
      },
      "dependsOn": [
        "[parameters('webAppName')]"
      ]
    }
```

Now we can use the same template to set different sets of restrictions for different App Services. If the IP ranges change, we have one location to modify the ranges and can simply deploy the resources again.