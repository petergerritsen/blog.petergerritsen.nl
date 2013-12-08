---
author: Peter Gerritsen
comments: true
date: 2010-06-26 13:08:23+00:00
layout: post
slug: set-default-value-on-a-managed-metadata-field-through-code
title: Set default value on a Managed Metadata field through code
wordpress_id: 806
tags:
- Managed Metadata
- SP2010
---

In some situations, such as metadata inheritance from web properties, you'll want to set the default value of a field from code. 
Because the DefaultValue property of a SPField is a string, you will need to know the correct format to set the value of a Managed Metadata column. The easiest way to retreive this value is to create an item with the correct value and read it with the following code: 

```csharp
SPList list = SPContext.Current.List;
SPField field = list.Fields["Thema"];
string value = field.GetValidatedString(SPContext.Current.ListItem[field.Id]);
```

When you use the resulting value as the default value on the field, everything should be okay. When you set an invalid default value, you will not be able to create new items anymore or set the default value through the UI. You can get around this by setting the default value back to an empty string.
