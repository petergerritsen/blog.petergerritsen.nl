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

After fiddling around for a bit I found the solution for setting it in a .NET core application ([based on a answer on stackoverflow](https://stackoverflow.com/questions/55927663/connect-to-azure-storage-queue-behind-proxy)). If you use the Microsoft.Azure.Cosmos.Table Nuget package, instead of the old WindowsAzure.Storage package (I'm using version 1.0.1), the `CloudTableClient` allows you to pass in a `TableClientConfiguration` with a `DelegatingHandler`:

```csharp
public TableStorage(string accountName, string keyValue, IWebProxy proxy)

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
```

In the DelegatingHandler you can set the proxy for the HttpClientHandler:

```csharp
public class StorageDelegatingHandler : DelegatingHandler
    {
        private readonly IWebProxy _proxy;

        private bool _firstCall = true;

        public StorageDelegatingHandler() 
            : base()
        {
        }

        public StorageDelegatingHandler(HttpMessageHandler httpMessageHandler)
            : base(httpMessageHandler)
        {
        }

        public StorageDelegatingHandler(IWebProxy proxy)
        {
            _proxy = proxy;
        }

        protected override Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, CancellationToken cancellationToken)
        {
            if (_firstCall && _proxy != null)
            {
                HttpClientHandler inner = (HttpClientHandler)InnerHandler;
                inner.Proxy = _proxy;
            }

            _firstCall = false;
            return base.SendAsync(request, cancellationToken);
        }
    }
```

Now you can configure the proxy where you're setting up dependency injection:

```csharp
public class ProxySettings
    {
        public bool Use => !string.IsNullOrWhiteSpace(Address);

        public string Address { get; set; }

        public bool BypassOnLocal { get; set; }
}
```

```csharp
var proxySettings = Configuration
                .GetSection(nameof(ProxySettings))
                .Get<ProxySettings>();
            IWebProxy proxy = null;
            if (proxySettings != null && proxySettings.Use)
            {
                proxy = new WebProxy(proxySettings.Address, proxySettings.BypassOnLocal);
                WebRequest.DefaultWebProxy = proxy;
            }

            services.AddSingleton<IStorage>(new TableStorage(Configuration["StorageAccountName"], Configuration["StorageAccountKey"], proxy));            
```

