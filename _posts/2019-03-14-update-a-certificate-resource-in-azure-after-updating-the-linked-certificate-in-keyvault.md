---
layout: post
comments: true
author: Peter
title: Update a certificate resource in Azure after updating the linked certificate
  in KeyVault
date: 2019-03-14 23:00:00 +0000
slug: update-certificate-resource-azure-after-renewing-linked-certificate-keyvault
wordpress_id: ''
tags:
- Resource Explorer
- Azure KeyVault
categories:
- Azure

---
After a certificate in Azure KeyVault is renewed, you might need to push it to the App Services that are using it. Certificates are stored in Azure as separate resources under the same resource group as the Azure Service Plan resides in. 

If you're using Infrastructure-as-Code (and you should) through ARM templates, you can redeploy the template that deploys the certificate resources. But there's also another way if you don't want to redeploy the templates (e.g. because it takes a long time depending on the amount of resources).

If you lookup the certificate through [Azure Resource Explorer](https://resources.azure.com) you can update the certificate though the UI. Just click the "Edit" button:![](/uploads/cert1.png)

Don't change anything to the request message and just click the "PUT" button. This will trigger Azure Resource manager to get the renewed certificate from Azure KeyVault and update it in the Service Plan.