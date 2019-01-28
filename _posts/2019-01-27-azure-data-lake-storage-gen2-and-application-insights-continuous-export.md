---
layout: post
comments: true
author: Peter
title: Azure Data Lake Storage Gen2 and Application Insights continuous export
date: 2019-01-27 23:00:00 +0000
slug: azure-data-lake-storage-gen2-and-application-insights-continuous-export
wordpress_id: ''
tags:
- Azure Data Lake
- Application Insights
categories: []

---
Summary: at the moment of writing, you cannot use Data Lake Storage Gen2 storage account as a target for Application Insights continuous export.

Currently Azure Data Lake Storage Gen2 is in preview. 

Because I want to use Big Data analytics on Application Insights information I've set up a pipeline through Stream analytics in the past to get the application insights data into Azure Data Lake. Now that Gen2 is available I wanted to see if you can export from Application Insights into Data Lake Storage Gen2 straight away, bypassing stream analytics or another component to transfer the data. So lets try:

From the portal, when creating a Storage account, you can specify you want to enable Data Lake Storage Gen2:

![](/uploads/azure-data-lake-storage-account-create-advanced.png)

Unfortunately, when setting up continuous export to this storage account, you cannot select a location, because creating a container is not possible:

![](/uploads/continuous-export-data-lake-gen2.PNG)

The exclamation mark indicates the container name is invalid, but it does match the requirements. 

When trying to create a container through Azure Storage Explorer, you get a more descriptive error:

![](/uploads/blob-container-hierarchical-storage.PNG)

So unfortunately, you cannot use continuous export to Data Lake Storage Gen2 right now. So we're still stuck with using stream analytics or Azure Data Factory.