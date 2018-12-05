---
title: ARM Template URL for LinkedTemplate stored in Blob storage
layout: post
comments: true
author: Peter
date: 2018-12-05 08:17:26 +0000
slug: arm-template-url-for-linkedtemplate-stored-in-blob-storage
wordpress_id: ''
tags: []
categories: []

---
At a customer we've set up an Infrastructure-As-Code solution with Azure Resource Manager (ARM) for deploying Azure Resources through a Azure DevOps build and release pipelines.

To use LinkedTemplates in ARM you need to provide a URI to the template in the calling template:

     "templateLink": { 
     	"uri":"https://mystorageaccount.blob.core.windows.net/AzureTemplates/newStorageAccount.json",           
        "contentVersion":"1.0.0.0"        
       }

This needs to be a location where Azure Resource Manager can download the template, without authentication. But what if you don't want to make your LinkedTemplates available for anyone? You can store them in a non public storage container, but use a SAS token. If you pass the URL plus SAS token to the calling template. It can use that to download the template. More about that [here](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-linked-templates#securing-an-external-template). 

We split all resource components up into separate templates, e.g. the ipSecurityRestrictions part for a Web App is in a separate template, which need to be called from the generic web app template we've created. I don't want to have to create parameters for the SAS token into each Linked Template I want to call. 

So I decided to use a different solution for creating the URI:

    "variables": {  
     "ipWhitelistingTemplateUrl": "[replace(deployment().properties.templateLink.uri, '/WebApp-Generic.json?', '/WebApp-IpWhitelisting.json?')]"
    }

It's not complicated and you can discuss if this is indeed a better solution. But I think it demonstrates the power of ARM templates. 