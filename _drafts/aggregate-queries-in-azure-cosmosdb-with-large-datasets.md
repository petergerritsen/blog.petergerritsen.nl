---
layout: post
comments: true
author: Peter
title: Aggregate queries in Azure CosmosDb with large datasets
date: 2020-02-18 23:00:00 +0000
slug: aggregate-queries-in-cosmosdb-with-large-datasets
wordpress_id: ''
tags:
- CosmosDb
- Azure
categories:
- Azure

---
One of the services I'm building at one of my customers is an API that provides invoice information for a customer self-service portal. The invoice information is stored (of course) in Azure CosmosDb. Invoices are partitioned by customerid, but hose partitions can still contain a lot of items. 

When querying for the total outstanding amount for not payed invoices we use an aggregate query:

    SELECT SUM(c.OutstandingAmount) AS TotalOutstandingAmount  FROM c WHERE c.Status <> 1

We execute the query with the following code: