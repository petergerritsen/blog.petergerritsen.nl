---
author: Peter Gerritsen
comments: true
date: 2009-02-03 16:55:08+00:00
layout: post
slug: retrieving-tasks-assigned-to-users-or-one-of-their-groups
title: Retrieving Tasks assigned to users or one of their groups
wordpress_id: 571
tags:
- CAML
- MOSS
categories: SharePoint
---

Sometimes you want to retrieve all tasks that are assigned to a user or to one of the SharePoint groups the user is a member of. This quite easy to accomplish through generation of a CAML query.

Just create an Or statement for tasks assigned to the user and loop through all of the groups in the SPUser object to add those to the query.

The resulting query will look like this:

```xml
Piet van Tul  
Region Controllers  
Accounting department  
```

By using this query in the QueryOverride of a Content Query Web Part you have all the power of a Content Query combined with the power of query generation.

EDIT: My collegue Wouter Lemaire pointed out that you can only specify 2 terms per Or or And element. So the above CAML could give some unexpected results.
