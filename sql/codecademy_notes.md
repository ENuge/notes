# Codecademy SQL Track

`JOIN` - will combine rows from different tables if the join condition is true.

`LEFT JOIN` - will return every row in the _left_ table, and if the join condition is not met, `NULL` values for all the right table columns.

_Foreign key_ is a column that contains the primary key to another table.

`CROSS JOIN` lets us combine all rows of one table with all rows of another table.

`UNION` stacks one dataset on top of another.

`WITH` allows us to define one or more temporary tables that can be used in the final query.

`AS` can be done with particular columns like so:
```sql
SELECT recipes.name AS 'Recipe',
  chefs.name AS 'Chef'
FROM recipes
JOIN chefs
  ON recipes.chef_id = chefs.id;
```

Here's an example group by, to get the average valuation per category:
```sql
SELECT category AS "Category",
  AVG(valuation) AS "Average valuation
FROM startups
GROUP BY 1;
```

You can combine `GROUP BY` and `ORDER BY` like so:
```sql
SELECT COUNT(CustomerID), Country
FROM Customers
GROUP BY Country
ORDER BY COUNT(CustomerID) DESC;
```

Can use `ROUND(..)` to round to a different amount of decimal places.

Use `HAVING` when you need to do a `WHERE` on an aggregate.

A relational database is a database that organizes information into one or more tables.

Here's an example of a `CASE` within the context of a whole lot of other columns:
```sql
SELECT name, neighborhood, cuisine,
  CASE
  	WHEN review > 4.5 THEN "Extraordinary"
    WHEN review > 4 THEN "Excellent"
    WHEN review > 3 THEN "Good"
    WHEN review > 2 THEN "Fair"
    ELSE "Poor" END AS "Categorized Reviews",
  price,
  health
FROM nomnom;
```

# SQL: Table Transformation

**Subqueries** allow you to nest one query within another query.

Example query with subquery present:
```sql
SELECT * 
FROM flights 
WHERE origin in (
    SELECT code 
    FROM airports 
    WHERE elevation > 2000);
```

^ That above is a non-correlated subquery - a subquery that can be run independently of the outer query.

You can perform subqueries on the same table, if you have like a multistep transformation for instance.
You can `ORDER BY` two columns, like: `ORDER BY 1,2;`

This will give you the average flight count per day of week each month:
```sql
SELECT a.dep_month,
       a.dep_day_of_week,
       AVG(a.flight_count) AS average_flights
  FROM (
        SELECT dep_month,
              dep_day_of_week,
               dep_date,
               COUNT(*) AS flight_count
          FROM flights
         GROUP BY 1,2,3
       ) a
 GROUP BY 1,2
 ORDER BY 1,2;
```

Now THIS is the sort of stuff I need to become a natural at!

**correlated subquery** - cannot be run independently of the outer query. First, you process a row in the outer query. Then, for that particular row, the subquery is executed. Here's an example:
```sql
SELECT id
FROM flights AS f
WHERE distance > (
 SELECT AVG(distance)
 FROM flights
 WHERE carrier = f.carrier);
```

Correlated subqueries can also appear in the `SELECT`. Here's an example:
```sql
SELECT carrier, id,
    (SELECT COUNT(*)
FROM flights f
WHERE f.id < flights.id
AND f.carrier=flights.carrier) + 1
 AS flight_sequence_number
FROM flights;
```

# Set Operations

Joins merge rows.
Unions merge columns.

Sample `UNION`:
```sql
SELECT column_name(s) FROM table1
UNION
SELECT column_name(s) FROM table2;
```

Each `SELECT` statement must have the same number of columns, in the same order, with similar data types. It by default only unions unique values.

`UNION ALL` allows us to get duplicate values.

`INTERSECT` returns only common rows returned by the two `SELECT` statements:
```sql
SELECT column_name FROM table1
INTERSECT
SELECT column_name FROM table2;
```

`EXCEPT` selects distinct rows from the first table that are not present in the second table:
```sql
SELECT column_name FROM table1
EXCEPT
SELECT column_name FROM table2;
```

# Conditional Aggregates

`ELSE` in `CASE` is optional, will default to `NULL`. Here's a sample `CASE` with an aggregate (note also the `BETWEEN` in here!):
```sql
SELECT
    CASE
        WHEN elevation < 500 THEN 'Low'
        WHEN elevation BETWEEN 500 AND 1999 THEN 'Medium'
        WHEN elevation >= 2000 THEN 'High'
        ELSE 'Unknown'
    END AS elevation_tier
    , COUNT(*)
FROM airports
GROUP BY 1;
```

Whoa, you can do a `CASE WHEN` from inside a `COUNT`!! Here's an example of that:
```sql
SELECT    state, 
    COUNT(CASE WHEN elevation >= 2000 THEN 1 ELSE NULL END) as count_high_elevation_aiports 
FROM airports 
GROUP BY state;
```

You can do that with `SUM(..)` too - note you want your case to not return 1 in the positive case, but to be the actual thing you're summing over:
```sql
SELECT origin, sum(distance) as total_flight_distance, sum(CASE WHEN carrier = 'UA' THEN distance ELSE 0 END) as total_united_flight_distance 
FROM flights 
GROUP BY origin;
```

You can combine aggregates to get things like percents or ratios pretty straightforwardly:
```sql
SELECT     origin, 
    100.0*(sum(CASE WHEN carrier = 'UN' THEN distance ELSE 0 END)/sum(distance)) as percentage_flight_distance_from_united FROM flights 
GROUP BY origin;
```

(I would probably split that percentage across a few lines ... -_-)

One big thing to note - integer division/truncation can screw you, if you don't keep the right order of operations so that the float'izing happens first!!

Also, use `NULL` in `COUNT`, not 0, as you would do for `SUM`!

# Date, Time, and String Functions

Dates are often written in one of two formats: YYYY-MM-DD or YYYY-MM-DD hh::mm::ss

This in SQLite (at least) would return the date and time for the `manufacture_time` column:
```sql
SELECT DATETIME(manufacture_time)
FROM baked_goods;
```

You can use the `DATE()` function to get just by-day info. For instance:
```sql
SELECT DATE(manufacture_time), count(*) as count_baked_goods
FROM baked_goods
GROUP BY DATE(manufacture_time);
```

And you can do the same with just `TIME()`:
```sql
SELECT TIME(manufacture_time), count(*) as count_baked_goods
FROM baked_goods
GROUP BY TIME(manufacture_time);
```

This lets you increment date or timestamp by a specified interval:
```sql
DATETIME(time1, '+3 hours', '40 minutes', '2 days');
```

You can do mathematical operations directly in the `SELECT` statements:
```sql
SELECT (number1 + number2);
```

Here's an example of type casting to avoid the integer division issue:
```sql
SELECT CAST(number1 AS REAL) / number3;
```

And for rounding to a specified number of decimals or sig figs:
```sql
SELECT ROUND(number, precision);
```

`MAX(..)` and `MIN(..)` respectively return the max+min values in the set of numeric expressions.

Concatenation of strings is done like so:
```sql
SELECT string1 || ' ' || string2;
```

For example:
```sql
SELECT city || ' ' || state as location
FROM bakeries;
```

You can replace all substrings with a given string like so:
```sql
REPLACE(string,from_string,to_string)
```

# SQL: Analyzing Business Metrics

What a shitty course. Lazy, bug-ridden, duplicative content. Like ~all of Codecademy's "created with" courses. Good riddance.