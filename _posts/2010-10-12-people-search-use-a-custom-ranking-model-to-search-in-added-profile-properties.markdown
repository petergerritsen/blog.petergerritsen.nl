---
author: Peter Gerritsen
comments: true
date: 2010-10-12 13:37:04+00:00
layout: post
slug: people-search-use-a-custom-ranking-model-to-search-in-added-profile-properties
title: 'People Search: Use a custom Ranking Model to search in added Profile Properties'
wordpress_id: 855
tags:
- Search
- SharePoint Profile System
- SP2010
---

When you use the out-of-the box components for searching for people, a generic search with a keyword that is included in an added Profile property will not give the results you might expect. 

This is especially a problem with the out-of-the-box webpart that shows information about a person:
  

[![](http://blog.petergerritsen.nl/wp-content/uploads/2010/10/Profile-properties-webpart-300x103.png)](http://blog.petergerritsen.nl/wp-content/uploads/2010/10/Profile-properties-webpart.png)
When you click on one of the values in this webpart, you will be redirected to the people search results page with the value you clicked passed in as the search keyword:
  

[![](http://blog.petergerritsen.nl/wp-content/uploads/2010/10/Profile-search-generic-300x105.png)](http://blog.petergerritsen.nl/wp-content/uploads/2010/10/Profile-search-generic.png)
But this will not give you any results. [This post](http://kgraeme.wordpress.com/2010/07/28/sharepoint-user-profile-custom-properties-keyword-search-problem/) by kgreame outlines the same problems and some of the steps he tried to solve this issue. In the comments to that post, a workaround is mentioned: map the matching crawled properties to the ContentsHidden Managed Property. 
There is however an other way. In [this post](http://sharepoint.microsoft.com/blogs/LKuhn/Lists/Posts/Post.aspx?List=29310d0a-1eda-4834-bb4c-06ee575a40c3&ID=52) by Larry Kuhn, he explains how the DEFAULTPROPERTIES within the SharePoint Search work. By setting a weight of a Managed Property, you will include it in the ranking model SharePoint uses and the property is therefore added to the default properties that are searched. 
SharePoint 2010 introduces the concept of Ranking Models, so the solution mentioned in that blogpost doesn't work. We can however create our own Ranking Model. I've copied the information from the Ranking Model that SharePoint uses for People Search and added my own Managed Property, Expertise, with a weight of 1.0:
```xml
<?xml version="1.0" encoding="utf-8"?>
<rankingModel name="CustomPeopleRanking" id="5EA2750C-8165-4f65-BD12-6E6DAAD45FE0" description="Custom People Ranking" xmlns="http://schemas.microsoft.com/office/2009/rankingModel">
  <queryDependentFeatures>
    <queryDependentFeature pid="177" name="RankingWeightName" weight="0.5" lengthNormalization="0" />
    <queryDependentFeature pid="19" name="PreferredName" weight="1.0" lengthNormalization="0" />
    <queryDependentFeature pid="24" name="JobTitle" weight="2.0" lengthNormalization="0" />
    <queryDependentFeature pid="39" name="Responsibilities" weight="1.0" lengthNormalization="5" />
    <queryDependentFeature pid="179" name="RankingWeightLow" weight="0.2" lengthNormalization="5" />
    <queryDependentFeature pid="175" name="ContentsHidden" weight="0.1" lengthNormalization="5" />
    <queryDependentFeature pid="35" name="Memberships" weight="0.25" lengthNormalization="5" />
    <queryDependentFeature pid="178" name="RankingWeightHigh" weight="2.0" lengthNormalization="0" />
    <queryDependentFeature pid="180" name="Pronunciations" weight="0.05" lengthNormalization="0" />
    <queryDependentFeature pid="408" name="Expertise" weight="1.0" lengthNormalization="0" />
  </queryDependentFeatures>
</rankingModel>
```
We can then add this Ranking Model to SharePoint with Powershell:
```
Get-SPEnterpriseSearchServiceApplication | New-SPEnterpriseSearchRankingModel
```
And paste in the XML of this Ranking Model when Powershell prompts you for it.

But how do we force SharePoint to use this Ranking Model? I've outlined one of the solutions in a [previous post](http://blog.petergerritsen.nl/2010/10/11/let-the-sharepoint-search-web-parts-use-an-other-ranking-model/). 

So after adding a web part that sets the ranking model we can perform the search again:  

[![](http://blog.petergerritsen.nl/wp-content/uploads/2010/10/Profile-search-after-ranking-model-300x139.png)](http://blog.petergerritsen.nl/wp-content/uploads/2010/10/Profile-search-after-ranking-model.png)
And bingo, the user with this value in an extra profile property shows up in the search results.

Why use this over mapping the properties in the 'ContentsHidden' Managed Property? As you can see the 'ContentsHidden' property is included in the Ranking Model with a value of 0.1. If you want to give more weight to a custom property or you have more properties to which you want to assign a different weight, you will need to modify the Ranking Model.


