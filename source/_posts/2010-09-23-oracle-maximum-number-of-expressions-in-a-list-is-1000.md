---
title: 'Oracle: maximum number of expressions in a list is 1000'
author: mike
layout: post
permalink: /2010/09/oracle-maximum-number-of-expressions-in-a-list-is-1000/
jabber_published:
  - 1285247988
categories:
  - IT
tags:
  - Oracle
---
“I have a comma delimited list of many of IDs that I want to use in an IN clause, Sometimes this list can exceed 1000 items, at which point Oracle throws an error. The query is similar to this&#8230;” – from <http://stackoverflow.com/questions/400255/how-to-put-more-than-1000-values-into-an-oracle-in-clause>

Recently I’ve encountered the same problem and I’m using Oracle11gR2.

Although it’s not recommended to have more than 1000 items in the IN clause list, we still need something to work around when sometimes it’s needed.

There might be some methods to work around this:

1. Wrap the method which would possibly make a query contains an IN list which exceed 1000 items to divide the large query into subqueries.

2. Using or in the where clause:

*select * from table1 where ID in (1,2,3,4,&#8230;,1000) or *

*ID in (1001,1002,&#8230;,2000)*

3. Using union all

*select * from table1 where ID in (1,2,3,4,&#8230;,1000)*

*union all*

*select * from table1 where ID in (1001,1002,&#8230;,2000)*

4. Temp table of course, the worst solution I think.

This problem does not appear to be in some of the other database systems.

If you’ve got some better solutions, please tell me.
