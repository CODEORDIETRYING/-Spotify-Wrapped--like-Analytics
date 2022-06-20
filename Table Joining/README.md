The rental table determines a lot in this analysis because it gives us information on each film rental which is what we want to analyze, but the rental table only shows us unique rental (rental_id), customer information (customer_id) and item rented (inventory_id). We do not know anything about the film in particular that was rented out as well as the film category. To know the category of the rented movie, we need to find a way to join the rental table to the category table. 

We can do this by following these steps:

- Step 1: Starting with the dvd_rental.Rental table, use the foreign key (inventory_id) to join it to the inventory table
- Step 2: Join the new table from step 1 to dvd_rental.film to get the film titles. The foreign key for the join is the film_id column
- Step 3: Join the dvd_rental.film table to the dvd_rental.film_category table on the film_id column
- Step 4: Finally join the previous table to the dvd_rental.category table on the category_id column serving as a foreign key

Here is a map of the join journey:

| S/N | Start | End | Foreign Key |
| ----------- | ----------- | ----------- | ----------- |
| Step 1 | rental | inventory | inventory_id |
| Step 2 | inventory | film | film_id |
| Step 3 | film | film_category | film_id |
| Step 4 | film_category | category | category_id |

## Table Relationship Hypothesis

Before joining tables together, it is best to understand the relationship between fields in a table and 
also the interrelationship with other tables in context with the use case required. Understanding this 
helps to know if a left join, inner join, cross join or full outer join is best suited. Here are some 
contextual hypotheses that I came up with before joining the tables together.

### Hypothesis 1 - Joining Rental and Inventory Tables

- The number of **unique** inventory_id records will be equal in both the dvd_rental.rental and dvd_rental.inventory tables. Meaning that all items in the inventory have been rented out in the past.
- There will be multiple records per unique inventory_id in the dvd_rentals.rental table. Meaning that a particular item with a unique inventory_id can be rented out multiple times.

Here is the SQL code to see if my hypothesis holds true. The first hypothesis failed because there was 
one more item in the inventory table than in the rental table, meaning that only one item was never 
rented out. Analysis from the tests also shows that there was no difference between running a left 
join from the rental table or an inner join.

To confirm this, here is a count of the number of rows and distinct key values for both types of joins.

```sql
DROP TABLE IF EXISTS left_rental_join;
CREATE TEMP TABLE left_rental_join AS
SELECT
  rental.customer_id,
  rental.inventory_id,
  inventory.film_id
FROM dvd_rentals.rental
LEFT JOIN dvd_rentals.inventory
  ON rental.inventory_id = inventory.inventory_id;

DROP TABLE IF EXISTS inner_rental_join;
CREATE TEMP TABLE inner_rental_join AS
SELECT
  rental.customer_id,
  rental.inventory_id,
  inventory.film_id
FROM dvd_rentals.rental
INNER JOIN dvd_rentals.inventory
  ON rental.inventory_id = inventory.inventory_id;

(
  SELECT
    'left join' AS join_type,
    COUNT(*) AS record_count,
    COUNT(DISTINCT inventory_id) AS unique_key_values
  FROM left_rental_join
)
UNION
(
  SELECT
    'inner join' AS join_type,
    COUNT(*) AS record_count,
    COUNT(DISTINCT inventory_id) AS unique_key_values
  FROM inner_rental_join
);
```

| Join_type | Record_count | Unique_key_count |
| ----------- | ----------- | ----------- |
| inner join | 16044 | 4580 |
| left join | 16044 | 4580 |

### Hypothesis 2 - Joining Inventory and Film Tables

- There will be multiple inventory_id records per unique film_id value in the dvd_rentals.inventory table.

A simple group by count with an additional summary group by will do to check this hypothesis.

We follow these 2 simple steps to summarise our dataset:

1. Perform a `GROUP BY` record count on the target column
2. Summarize the record count output to show the distribution of records by unique count of the target column

The target column in this case is our `inventory_id` field.

<details>
  <summary>You can find the SQL code for this check here.</summary>

```sql
    -- first generate group by counts on the target_column_values column
    WITH counts_base AS (
    SELECT
      inventory_id AS target_column_values,
      COUNT(*) AS row_counts
    FROM dvd_rentals.rental
    GROUP BY target_column_values
    )
    -- summarize the group by counts above by grouping again on the row_counts from counts_base CTE part
    SELECT
      row_counts,
      COUNT(target_column_values) as count_of_target_values
    FROM counts_base
    GROUP BY row_counts
    ORDER BY row_counts;
```

</details>
    
    

From the analysis we see that there is also no difference between a left and inner join implementation as proven below:

```sql
DROP TABLE IF EXISTS left_join_part_2;
CREATE TEMP TABLE left_join_part_2 AS
SELECT
  inventory.inventory_id,
  inventory.film_id,
  film.title
FROM dvd_rentals.inventory
LEFT JOIN dvd_rentals.film
  ON film.film_id = inventory.film_id;

DROP TABLE IF EXISTS inner_join_part_2;
CREATE TEMP TABLE inner_join_part_2 AS
SELECT
  inventory.inventory_id,
  inventory.film_id,
  film.title
FROM dvd_rentals.inventory
LEFT JOIN dvd_rentals.film
  ON film.film_id = inventory.film_id;

(
  SELECT
    'left join' AS join_type,
    COUNT(*) AS record_count,
    COUNT(DISTINCT film_id) AS unique_key_values
  FROM left_join_part_2
)
UNION
(
  SELECT
    'inner join' AS join_type,
    COUNT(*) AS record_count,
    COUNT(DISTINCT film_id) AS unique_key_values
  FROM inner_join_part_2
);
```

| Join Type | Record_count | Unique_key_count |
| ----------- | ----------- | ----------- |
| inner-join | 4581 | 958 |
| left-join | 4581 | 958 |


## Table Join

For the categories section of the case study, we do not need any information on the actors, 
all our analysis stops at finding movie recommendations and that means we can stop the table 
joins on the **dvd_rental.category** dataset.

Here is the code for joining all five tables together:

```sql
DROP TABLE IF EXISTS complete_joint_dataset;
CREATE TEMP TABLE complete_joint_dataset AS
SELECT
  rental.customer_id,
  inventory.film_id,
  film.title,
  film_category.category_id,
  category.name AS category_name
FROM dvd_rentals.rental
INNER JOIN dvd_rentals.inventory
  ON rental.inventory_id = inventory.inventory_id
INNER JOIN dvd_rentals.film
  ON inventory.film_id = film.film_id
INNER JOIN dvd_rentals.film_category
  ON film.film_id = film_category.film_id
INNER JOIN dvd_rentals.category
  ON film_category.category_id = category.category_id;

SELECT * FROM complete_joint_dataset limit 2;
```

### Result
| customer_id | film_id | title | category_id | category |
| ----------- | ----------- | ----------- | ----------- | ----------- |
| 130 | 80 | BLANKET BEVERLY | 8 | Family |
| 459 | 333 | FREAKY POCUS | 12 | Music |
