+++
date = "2017-06-19T13:37:56+02:00"
#subtitle = "Examples of nested queries"
tags = []
title = "SQL - Find duplicate values | Snippets"

+++

Find multiple entries inside a table for given values.
This example return users which share their email with other users.

```sql
SELECT
    name, email, COUNT(*)
FROM
    users
GROUP BY
    name, email
HAVING 
    COUNT(*) > 1
```
