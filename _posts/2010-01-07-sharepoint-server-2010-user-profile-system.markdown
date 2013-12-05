---
author: admin
comments: true
date: 2010-01-07 09:47:00+00:00
layout: post
slug: sharepoint-server-2010-user-profile-system
title: SharePoint Server 2010 User Profile System
wordpress_id: 667
tags:
- SharePoint Profile System
- SP2010
---

In SharePoint 2010 there are quite a few interesting changes to the User Profile System. In this post I will outline some of them. 

 

### Profile types

 

You can specify sub-types for profiles. For each profile property you can configure for which types the property is used. A user will be able to see or edit only those properties that are linked to their Profile type:

 

[![image](http://blog.petergerritsen.nl/wp-content/uploads/image_thumb.png)](http://blog.petergerritsen.nl/wp-content/uploads/image.png)

 

In the Profile list or Profile property list you can filter the list by profile sub-type:

 

[![image](http://blog.petergerritsen.nl/wp-content/uploads/image_thumb1.png)](http://blog.petergerritsen.nl/wp-content/uploads/image1.png)

 

### Organization profiles

 

You can now import organizations from your profile store. This includes options for importing organizations in a hierarchy. The organization profile system also supports specification of Profile Types, so different types of organizations can have different properties.

 

 

 

 

[![image](http://blog.petergerritsen.nl/wp-content/uploads/image_thumb2.png)](http://blog.petergerritsen.nl/wp-content/uploads/image2.png)

 

 

 

 

The hierarchy is specified by selecting a parent organization in an organization profile. You can also specify the leaders of an organization and the members of that organization. This will link the specified user profiles to this organization:

 

[![image](http://blog.petergerritsen.nl/wp-content/uploads/image_thumb3.png)](http://blog.petergerritsen.nl/wp-content/uploads/image3.png)

 

### Property Synchronization

 

Profile properties can now be exported as well. This allows for users to edit a property value which is then updated in your identity store such as Active Directory:

 

[![image](http://blog.petergerritsen.nl/wp-content/uploads/image_thumb4.png)](http://blog.petergerritsen.nl/wp-content/uploads/image4.png)

 

This sync is one way only, so a property can be imported or exported, but there’s no option to keep the two values in sync in a bi-directional way. Export to BCS sources is also not supported.

 

### Term store used for choices

 

Properties that are defined with choices are linked to the Term store. So there’s a single management system for choice fields across the SharePoint farm:

 

[![image](http://blog.petergerritsen.nl/wp-content/uploads/image_thumb5.png)](http://blog.petergerritsen.nl/wp-content/uploads/image5.png)

 

The property will use the configuration of the Term set to define if users are allowed to use fill-in choices.

 

### Multiple import connections

 

In SharePoint 2007 you were only able to import from one primary store. So importing from 2 or more Active Directories or a custom user database was not supported. In 2010 this is supported. You can now specify more then 1 import connection to import accounts from AD as well as a LDAP or BCS store.

 

### Import filters

 

You can specify exclusion filters for your import connections. This will allow you to filter out user or organization profiles when importing from you identity store:

 

[![image](http://blog.petergerritsen.nl/wp-content/uploads/image_thumb6.png)](http://blog.petergerritsen.nl/wp-content/uploads/image6.png)

 

### Conclusion

 

The SharePoint team has done quite a big overhaul on the user profile system. A lot of pain points from MOSS 2007 have been solved, the system allows for more granualarity in configuration of the profiles and has been integrated nicely with new functionality such as the Term store (a.k.a. Managed Metadata Service)
