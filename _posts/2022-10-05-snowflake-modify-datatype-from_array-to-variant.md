---
layout:  post
title:   Snowflake modify data type from array to variant
date:   2022-10-09
summary:  This blog post explains step-by-step procedures about how to change data type without losing existing data in snowflake.
categories: blogs
---

Sometimes we need to modify the data type of a database table attribute due to a change in the data structure 
or due to other reasons.

Here we will see how to alter the column from array to variant without losing existing data with a scenario.

Suppose we have a column `order_ids` with a data type array in the table `user_orders`.

|id| order_ids |
| - | - |
|1 | ["7","2","3] |
|2 | ["10","5","4"] |
|3 | ["14","8","30"] |

And as per the new requirement, we need to store data in JSON format.

|id| order_ids|
|-|-|
|1| [ { "order_id": "7" }, { "order_id": "2" }, { "order_id": "3" } ] |
|2| [ { "order_id": "10" }, { "order_id": "5" }, { "order_id": "4" } ] |
|3| [ { "order_id": "14" }, { "order_id": "8" }, { "order_id": "30" } ] |


Modifying data type works in a few cases, like -incrementing varchar size.

However, decrement varchar won't work. Similarly, Array to Variant will raise an error.

```sql
alter table user_orders modify order_ids set data type variant;
```

This `alter` command will raise the below error-

```sql
SQL compilation error: Can not change column order_ids
from type ARRAY to VARIANT
```

### Solution
Below are the 4 steps to fix the issue -

1\. make a backup of the order_ids

We are going to rename `order_ids` to `order_ids_backup`

```sql
alter table user_orders rename column order_ids to order_ids_backup;
```

2\. Now, as we have renamed `order_ids` column and hence we can recreate it with data type `VARIANT`

```sql
alter table user_orders add order_ids VARIANT;`
```

3\. After this, we need to push existing data with the new format from the backup attribute.

```sql 
UPDATE user_orders
SET order_ids = PARSE_JSON(regexp_replace(regexp_replace(regexp_replace(cast(order_ids_backup as string),'\\["','[{"order_id":"'),'","','"},{"order_id":"'), '\\"]','"}]'));
```

Let's try to understand what we did inside PARSE_JSON—

 - cast(order_ids_backup as string)
 This will change the type from array to string. This will help us to create a new JSON format as we discussed above.
 - Then we have 3 regular expressions to replace and create a format.
  - first replace `'\\["'` to `'[{"order_id":"'`
  - then `'","'` to `'"},{"order_id":"'`
  - Lastly, `'\\"]'`,`'"}]'`

4\. Drop the renamed(backup) column as we have copied the data to `order_ids` column.
```sql
alter table user_orders drop column order_ids_backup;
```

## Summary

There are four simple steps to changing data types without losing data:
- Step 1: Rename the existing column (order_ids to order_ids_backup).
- Step 2: Add a new column with the old column name(order_ids) to which you want to change the data type to.
- Step 3: Update new column value(order_ids) with renamed column value(order_ids_backup)
- Step 4: Drop renamed column(order_ids_backup)

If you have any other suggestions, please let me know in the comment section.
