---
author: Peter Gerritsen
comments: true
date: 2015-11-07 10:00:00+00:00
layout: post
slug: determine-first-upcoming-date-based-on-interval-in-t-sql
title: Determine first upcoming date based on an interval in T-SQL
tags:
- SQL Server 2012
- T-SQL
categories: SQL
---

For a project we needed to determine the first upcoming date of items based on a start date and an interval in a SQL query.
So the interval would be a number of years, months, weeks or days. 

For this, I decided to create a scalar function that would calculate the next upcoming date. 

We can offcourse use the DATEADD function, but we need the correct amount of years, months etc. to add to the startdate.

This will be the interval times the difference in years, months etc. divided by the interval. We need to add this one more time if the date for this year is already passed.

```sql
-- =============================================
-- Author:          <Author,,Name>
-- Create date: <Create Date, ,>
-- Description:     <Description, ,>
-- =============================================
CREATE FUNCTION [dbo].[fnGetNextDateFromDateAndInterval]
(
       -- Add the parameters for the function here
       @firstDate DATETIME,
       @numberOfYears INT,
       @numberOfMonths INT,
       @numberOfWeeks INT,
       @numberOfDays INT
)
RETURNS DATETIME
AS
BEGIN
       IF @numberOfYears IS NOT NULL
       BEGIN
             RETURN DATEADD(YEAR, @numberOfYears * ((DATEDIFF(YEAR, @firstDate, GETDATE()) / @numberOfYears) + CASE WHEN DATEFROMPARTS(YEAR(GETDATE()), MONTH(@firstDate), DAY(@firstDate)) > GETDATE() THEN 0 ELSE 1 END), @firstDate)
       END
 
       IF @numberOfMonths IS NOT NULL
       BEGIN
             RETURN DATEADD(MONTH, @numberOfMonths * ((DATEDIFF(MONTH, @firstDate, GETDATE()) / @numberOfMonths) + CASE WHEN DATEFROMPARTS(YEAR(GETDATE()), MONTH(@firstDate), DAY(@firstDate)) > GETDATE() THEN 0 ELSE 1 END), @firstDate)
       END
 
       IF @numberOfWeeks IS NOT NULL
       BEGIN
             RETURN DATEADD(WEEK, @numberOfWeeks * ((DATEDIFF(WEEK, @firstDate, GETDATE()) / @numberOfWeeks) + CASE WHEN DATEFROMPARTS(YEAR(GETDATE()), MONTH(@firstDate), DAY(@firstDate)) > GETDATE() THEN 0 ELSE 1 END), @firstDate)
       END
 
       IF @numberOfDays IS NOT NULL
       BEGIN
             RETURN DATEADD(DAY, @numberOfDays * ((DATEDIFF(WEEK, @firstDate, GETDATE()) / @numberOfDays) + CASE WHEN DATEFROMPARTS(YEAR(GETDATE()), MONTH(@firstDate), DAY(@firstDate)) > GETDATE() THEN 0 ELSE 1 END), @firstDate)
       END
 
       RETURN GETDATE()
END
 
GO
```

To make sure this function calculates the correct date, I decided to create some unit tests. I like to use [Dapper](https://github.com/StackExchange/dapper-dot-net) to perform any DB queries against SQL server.

```csharp
using System;
using Microsoft.VisualStudio.TestTools.UnitTesting;
using System.Data.SqlClient;
using Dapper;
using System.Data;
using System.Linq;

namespace TestDbFunctionsCSharp
{
    [TestClass]
    public class UnitTest1
    {
        [TestMethod]
        public void TestJaren()
        {
            using (SqlConnection conn = new SqlConnection("CONNSTRING_HERE"))
            {
                conn.Open();

                var refDate = DateTime.Today.AddDays(2);

                // Jaren
                Assert.AreEqual(refDate, GetNextDate(conn, refDate.AddYears(9 * -3), 3));               
            }
        }

        [TestMethod]
        public void TestMaanden()
        {
            using (SqlConnection conn = new SqlConnection("CONNSTRING_HERE"))
            {
                conn.Open();

                Assert.AreEqual(DateTime.Today.AddMonths(4), GetNextDate(conn, DateTime.Today.AddMonths(-8), numberOfMonths: 6));

                Assert.AreNotEqual(DateTime.Today, GetNextDate(conn, DateTime.Today.AddMonths(-7), numberOfMonths: 1));

                // Weken
                //Assert.AreEqual(DateTime.Today, GetNextDate(conn, DateTime.Today.AddDays(-7 * 8), numberOfWeeks: 2));
            }
        }

        [TestMethod]
        public void TestWeken()
        {
            using (SqlConnection conn = new SqlConnection("CONNSTRING_HERE"))
            {
                conn.Open();

                Assert.AreEqual(DateTime.Today.AddDays(14), GetNextDate(conn, DateTime.Today.AddDays(-7 * 8), numberOfWeeks: 2));

                Assert.AreEqual(DateTime.Today.AddDays(14), GetNextDate(conn, DateTime.Today.AddDays(-7 * 80), numberOfWeeks: 2));

                Assert.AreEqual(DateTime.Today.AddDays(7), GetNextDate(conn, DateTime.Today.AddDays(-7 * 79), numberOfWeeks: 2));
            }
        }

        private DateTime GetNextDate(SqlConnection conn, DateTime firstDate, int? numberOfYears = null, int? numberOfMonths = null, int? numberOfWeeks = null, int? numberOfDays = null)
        {
            var retval = conn
                    .Query<DateTime>(
                        "SELECT dbo.fnGetNextDateFromDateAndInterval(@firstDate, @numberOfYears, @numberOfMonths, @numberOfWeeks, @numberOfDays) AS VolgendeDatum",
                        new
                        {
                            firstDate,
                            numberOfYears,
                            numberOfMonths,
                            numberOfWeeks,
                            numberOfDays
                        },
                        commandType: CommandType.Text
                    ).First();

            return retval;
        }
    }
}
```

