---
layout: post
comments: true
author: Peter
title: Retrieve hybrid connection usage with Azure CLI / PowerShell Core
date: 2020-02-18 23:00:00 +0000
slug: retrieve-hybrid-connection-usage-azure-cli-powershell-core
wordpress_id: ''
tags:
- Hybrid connections
- Azure CLI
- PowerShell
categories:
- Azure

---

At a customer we make use of Hybrid Connections to connect App Services to on-premise applications. Today I was working
on changing those hybrid connections as we are migrating the on-premise services which involves changing the hostnames.

I was cleaning up old connections but the amount of used connections in the App Service plan wouldn't come down. We were
hitting the limit of 25 connections, so I wasn't able to add new ones for the new hostnames.

Obviously some connections were still in use in other App Services I wasn't aware of. And there are a lot of web apps 
in this subscription, which I didn't want to check one-by-one. So I needed an easy way to list the
Hybrid connections in the subscription and where they were used. 

First I tried Azure Resource Graph Explorer to find the used Hybrid connections, but unfortunately that doesn't contain
the child resources on the App Services which I was looking for. So I resorted to Azure CLI and PowerShell Core to do 
the job.

First, make sure you are logged in with Azure CLI and then list all the webapps in the subscription you want to query.
We let Azure CLI output a json array of objects with the webapp name and resourceGroup as property and then pipe it to 
`ConvertFrom-Json`.  

```PowerShell
az login
$subscriptionId = '11111111-1111-1111-1111-111111111111'

$sites = az webapp list --subscription $subscriptionId --query "[].{WAName:name, WARg:resourceGroup}" -o json | ConvertFrom-Json
``` 

Then we can loop through all the sites and retrieve the linked Hybrid connections. We add the Hybrid connection to a 
hashset with the name of the webapp in an array as it value. If the hybrid connection is already added to the hashset
before we add the name of the webapp to the value:

```PowerShell
$h = @{}

foreach ($site in $sites)
{
    $hybridConnections = az webapp hybrid-connection list --name $site.WAName --resource-group $site.WARg --subscription $subscriptionId | ConvertFrom-Json
    
    foreach($hybridConnection in $hybridConnections){
        if ($h.ContainsKey($hybridConnection.name)){
            $h[$hybridConnection.name] += $site.WAName
        }
        else {
            $h.Add($hybridConnection.name, @($site.WAName))
        }
    }    
}
```

Then we can write out all the connections and the apps they are used in:

```PowerShell
foreach ($key in $h.keys)
{
    Write-Host $key
    foreach ($webapp in $h[$key])
    {
        Write-Host "  -- $webapp"
    } 
}
```

So a little bit for PowerShell and Azure CLI goodness can help you if you have a lot of webapps you don't want to check
one-by-one.