---
layout: post
comments: true
author: Peter
title: Configure proxy for Azure Storage / CloudTableClient in .NET Core
date: 2019-06-04 22:00:00 +0000
slug: configure-proxy-for-azure-storage-cloudtableclient
wordpress_id: ''
tags:
- Azure Storage
categories:
- Azure

---
For a project I'm working on I needed to specify a outgoing proxy for accessing Azure Table Storage in a .NET console application.

Unfortunately the default way of setting a proxy in the app.config of classic .NET applications doesn't work for .NET core.

After fiddling around for a bit I found the solution for setting it in a .NET core application. The `CloudTableClient` allows you to pass in a `TableClientConfiguration`with a `DelegationHandler`:

\`\`\`csharp

public BlobStorage(string accountName, string keyValue, IWebProxy proxy)

        {

            _storageAccount =

                new CloudStorageAccount(

                    new StorageCredentials(accountName, keyValue), true);

            var storageDelegatingHandler = new StorageDelegatingHandler(proxy);

            _tableClient = _storageAccount.CreateCloudTableClient(new TableClientConfiguration

            {

                RestExecutorConfiguration = new RestExecutorConfiguration

                {

                    DelegatingHandler = storageDelegatingHandler

                }

            });

/// further config

\`\`\`