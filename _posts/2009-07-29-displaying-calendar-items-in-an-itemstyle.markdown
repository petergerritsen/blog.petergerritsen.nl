---
author: Peter Gerritsen
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


```
xmlns:ddwrt="http://schemas.microsoft.com/WebParts/v2/DataView/runtime"
```

And finally here’s the itemstyle:

'''xml
<xsl:template name="CalendarEvent" match="Row[@Style='CalendarEvent']" mode="itemstyle">
     <xsl:variable name="SafeImageUrl">
        <xsl:call-template name="OuterTemplate.GetSafeStaticUrl">
            <xsl:with-param name="UrlColumnName" select="'ImageUrl'"/>
        </xsl:call-template>
    </xsl:variable>
    <xsl:variable name="SafeLinkUrl">
        <xsl:call-template name="OuterTemplate.GetSafeLink">
            <xsl:with-param name="UrlColumnName" select="'LinkUrl'"/>
        </xsl:call-template>
    </xsl:variable>
    <xsl:variable name="DisplayTitle">
        <xsl:call-template name="OuterTemplate.GetTitle">
            <xsl:with-param name="Title" select="@Title"/>
            <xsl:with-param name="UrlColumnName" select="'LinkUrl'"/>
        </xsl:call-template>
   </xsl:variable>
      <xsl:variable name="LinkTarget">
         <xsl:if test="@OpenInNewWindow = 'True'" >_blank</xsl:if>
   </xsl:variable>
   <xsl:variable name="MultiDayEvent">
       <xsl:choose>
           <xsl:when test="starts-with(@EndDate,substring(@EventDate, 0, 11))">
               0
           </xsl:when>
           <xsl:otherwise>
               1
           </xsl:otherwise>
       </xsl:choose>
   </xsl:variable>
   <xsl:variable name="StartTimeIsEndTime">
       <xsl:choose>
           <xsl:when test="contains(@EndDate,substring(@EventDate, 11, 9))">
               1
           </xsl:when>
           <xsl:otherwise>
               0
           </xsl:otherwise>
       </xsl:choose>
   </xsl:variable>
   <xsl:variable name="DisplayDate">
       <xsl:choose>
           <xsl:when test="$MultiDayEvent = 0">
               <xsl:choose>
                   <xsl:when test="@fAllDayEvent = 0">
                    <xsl:choose>
                           <xsl:when test="$StartTimeIsEndTime = 1">
                               <xsl:value-of select="ddwrt:FormatDateTime(string(@EventDate) ,1043 ,'dd-MM-yyyy H:mm')" />
                           </xsl:when>
                           <xsl:otherwise>
                               <xsl:value-of select="ddwrt:FormatDateTime(string(@EventDate) ,1043 ,'dd-MM-yyyy H:mm')" /> - <xsl:value-of select="ddwrt:FormatDateTime(string(@EndDate) ,1043 ,'H:mm')" />
                           </xsl:otherwise>
                       </xsl:choose>
                   </xsl:when>
                   <xsl:otherwise>
                    <xsl:value-of select="ddwrt:FormatDateTime(string(@EventDate) ,1043 ,'dd-MM-yyyy')" />
                </xsl:otherwise>
               </xsl:choose>
           </xsl:when>
           <xsl:otherwise>
               <xsl:choose>
                   <xsl:when test="@fAllDayEvent = 0">
                       <xsl:value-of select="ddwrt:FormatDateTime(string(@EventDate) ,1043 ,'dd-MM-yyyy H:mm')" /> - <xsl:value-of select="ddwrt:FormatDateTime(string(@EndDate) ,1043 ,'dd-MM-yyyy H:mm')" />
                   </xsl:when>
                   <xsl:otherwise>
                       <xsl:value-of select="ddwrt:FormatDateTime(string(@EventDate) ,1043 ,'dd-MM-yyyy')" /> - <xsl:value-of select="ddwrt:FormatDateTime(string(@EndDate) ,1043 ,'dd-MM-yyyy')" />
                   </xsl:otherwise>
               </xsl:choose>
           </xsl:otherwise>
       </xsl:choose>
   </xsl:variable>
   <a href="{$SafeLinkUrl}" target="{$LinkTarget}" title="{@LinkToolTip}">
        <xsl:value-of select="$DisplayTitle"/>
    </a>
    <xsl:text> - </xsl:text><xsl:value-of select="$DisplayDate"/><br/>
</xsl:template>
```
