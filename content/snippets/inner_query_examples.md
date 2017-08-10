+++
date = "2017-06-19T13:37:56+02:00"
#subtitle = "Examples of nested queries"
tags = []
title = "Nested SELECT query | Snippets"

+++

How to use inner select query in SQL.

```sql
SELECT t.date , COUNT(*) AS player_count
FROM (
    SELECT DATE(MIN(`date`)) AS date
    FROM player_playtime
    GROUP BY player_name
) AS t
GROUP BY t.date DESC LIMIT 60 ;
```
