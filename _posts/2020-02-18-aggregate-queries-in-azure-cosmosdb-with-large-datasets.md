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
One of the services I'm building at one of my customers is an API that provides invoice information for a customer self-service portal. The invoice information is stored (of course) in Azure CosmosDb. Invoices are partitioned by customerid, but those partitions can still contain a lot of items.

When querying for the total outstanding amount for not payed invoices you can use an aggregate query:

```sql
SELECT SUM(c.OutstandingAmount) AS TotalOutstandingAmount  FROM c WHERE c.Status <> 1
``` 

We executed the query with the following code:

```csharp
var feedOptions = new FeedOptions
{
    EnableCrossPartitionQuery = false,
    PartitionKey = new PartitionKey(partitionKey)
};

var querySpec = new SqlQuerySpec() 
{ 
    QueryText = queryText, 
    Parameters = new SqlParameterCollection(queryParameters.Select(pair => new SqlParameter(pair.Key, pair.Value))) 
};

using( var query = _documentClient.Value.CreateDocumentQuery<T>(collectionUri, querySpec, feedOptions).AsDocumentQuery())
{
    var response = await query.ExecuteNextAsync<T>(cancellationToken);
} 

return response.First();
```

But sometimes this returned an item with an amount set to 0 where I knew this should not be the case. When I ran the same code against a test database, the issue did not arise.

So I resorted to Fiddler to help me find the issue between the two queries.

First I tried running the query against the test database:

![](/uploads/CosmosAggregateTST.PNG)

And then against the acceptation database:

![](/uploads/CosmosAggregateACC.PNG)

As you can see, the latter returns a continuation token in the response through a response header. So this means we should continue asking for results in our code:

```csharp
var items = new List<T>();

using (var query = _documentClient.Value.CreateDocumentQuery<T>(collectionUri, querySpec, feedOptions).AsDocumentQuery())
{
    while (query.HasMoreResults)
    {
        var response = await query.ExecuteNextAsync<T>(cancellationToken);
        
        items.AddRange(response);
    }
}

return items;
```
    
Then you can create an aggregated result by using Linq to sum up the values of the returned items.