---
author: admin
comments: true
date: 2009-07-29 14:05:21+00:00
layout: post
slug: displaying-calendar-items-in-an-itemstyle
title: Displaying calendar items in an ItemStyle
wordpress_id: 547
tags:
- MOSS
- XSLT
---


We frequently get the question to display the next _x_ upcoming events in a rollup webpart. A content query web part is very suitable for this, but you will needa custom ItemStyle. Below you will find the xsl to display events in the following
format:






[![image](http://blog.petergerritsen.nl/wp-content/uploads/snipping17.png)](http://blog.petergerritsen.nl/wp-content/uploads/snipping16.png)






The custom itemstyle checks for the difference between all day events, events that span multiple days and checks for start and end times.






To use this itemstyle you will need to add EventDate and EndDate to the CommonViewFields and reference the ddwrt namespace in the top of your xsl:



[sourcecode language="xslt"]xmlns:ddwrt="http://schemas.microsoft.com/WebParts/v2/DataView/runtime"[/sourcecode]



And finally here’s the itemstyle:



[sourcecode language="xslt"]

         
            
                
            
        
        
            
                
            
        
        
            
                
                
            
       
          
             _blank
       
       
           
               
                   0
               
               
                   1
               
           
       
       
           
               
                   1
               
               
                   0
               
           
       
       
           
               
                   
                       
                        
                               
                                   
                               
                               
                                    - 
                               
                           
                       
                       
                        
                    
                   
               
               
                   
                       
                            - 
                       
                       
                            - 
                       
                   
               
           
       
       [
            
        ]({$SafeLinkUrl})
         -   

    
[/sourcecode]
