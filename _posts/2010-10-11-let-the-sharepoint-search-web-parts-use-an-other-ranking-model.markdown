---
author: Peter Gerritsen
comments: true
date: 2010-10-11 18:48:47+00:00
layout: post
slug: let-the-sharepoint-search-web-parts-use-an-other-ranking-model
title: Let the SharePoint Search web parts use an other Ranking Model
wordpress_id: 850
tags:
- Search
- SP2010
categories: SharePoint
---

SharePoint 2010 introduces the concept of Ranking Models. These models allow you to control the ranking of search results.

But how do you tell the search web parts to use your custom ranking model, once you specified it? As far as I can tell there’s 2 options:

  * Pass in the id of the model through the querystring by appending `&rm=<guid of ranking model>`
  * Use the Shared Query Manager

The Shared Query Manager is _the_ option for modifying the query the search web parts are going to use.

To use the Query Manager to set an other Ranking Model I’ve created a custom web part. To retrieve the Query Manager all you have to do is call the static GetInstance method of the SharedQueryManager class.

After that, you can loop through the query manager and each location within the LocationList to set the ID of the Ranking Model:

```csharp
public class SetRankingModel : WebPart
    {
        protected override void OnInit(EventArgs e)
        {
            try
            {
                QueryManager qm = SharedQueryManager.GetInstance(this.Page).QueryManager;
                foreach (LocationList ll in qm)
                {
                    foreach (Location l in ll)
                    {
                        try
                        {
                            l.RankingModelID = "5ea2750c-8165-4f65-bd12-6e6daad45fe1";
                        }
                        catch { }
                    }
                }
                base.OnInit(e);
            }
            catch { }
        }
    }
```

After you’ve placed this web part on a search results page, each web part will use your custom Ranking Model.
