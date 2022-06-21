We have now joined 5 tables together needed to help the marketing team with the first part of the analysis. 
Let’s take another look at the base table.

<details>
  <summary>Base Table (complete_joint_dataset)</summary>

  ```sql
  DROP TABLE IF EXISTS complete_joint_dataset;
CREATE TEMP TABLE complete_joint_dataset AS
SELECT
  rental.customer_id,
  inventory.film_id,
  film.title,
  rental.rental_date,
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

SELECT * FROM complete_joint_dataset limit 5;
  ```

</details>

| customer_id | film_id | title | rental_date | category |
| ----------- | ----------- | ----------- | ----------- | ----------- |
| 130 | 80 | BLANKET BEVERLY | 2005-05-24 22:53:30 | Family |
| 459 | 333 | FREAKY POCUS | 2005-05-24 22:54:33 | Music |
| 408 | 373 | GRADUATE LORD | 2005-05-24 23:03:39 | Children |
| 333 | 535 | LOUD SUICIDES | 2005-05-24 23:04:41 | Horror |
| 222 | 450 | IDOLS SNATCHERS | 2005-05-24 23:05:21 | Children |

We saw earlier that the email template can be arranged into 5 sections:

<details>
<summary>Requirement 1: Top 2 Categories</summary>
  
 ![Top Categories](https://raw.githubusercontent.com/CODEORDIETRYING/Marketing-Analytics-Case-Study/main/Images/Top%202%20categories%20circled.PNG)
  
 **Data Point:** category_name
</details>

<details>
<summary>Requirement 2&3: Category Insights</summary>
  
 ![Category Insights](https://raw.githubusercontent.com/CODEORDIETRYING/Marketing-Analytics-Case-Study/main/Images/Top%202%20categories%20insights%20circled.PNG)
  
 Data Points: rental_count, category_name, average_comparison, percentile, category_percentage
</details>

<details>
<summary>Requirement 4: Category Film Recommendations</summary>
  
 ![Category Recommendations](https://raw.githubusercontent.com/CODEORDIETRYING/Marketing-Analytics-Case-Study/main/Images/top%202%20categories%20recommendations.PNG)
  
</details>

<details>
<summary>Requirement 5: Favorite Actor & Recommendations</summary>
  
 ![Category Recommendations](https://github.com/CODEORDIETRYING/Marketing-Analytics-Case-Study/blob/main/Images/Favourite%20actor%20and%20recommendation%20circled.PNG?raw=true)
  
 **Data Points:** actor_name, top_actor_counts
</details>


When we look into the data points that make up each section, we then came up with the following data 
points:

- `category_name`: The name of the top and second-ranking categories
- `category_rental_count`: There are two category rental counts here for each of the top 2 categories
- `average_comparison`: How many more films has the customer watched compared to the average LetFlix customer in their top categories? For this, we would need to count the number of times a customer has rented a film in their top category and subtract the number from the average count or rentals across that category.
- `percentile`: How does the customer rank in terms of the top X% compared to all other customers in this film category?
- `category_percentage`: What proportion of each customer’s total films watched does this count make?
- `actor_name`: The full name of the customers’ top actor
- `top_actor_counts`: The number of times a customer has rented a movie featuring their top actor.

---
<details>
<summary>Requirements 1, 2 & 3: Top Categories & Insights</summary>
1. `category_name`, `category_rental_count`:

We are going to be manipulating the base dataset to arrive at each datapoint. To begin, we will need to find the rental count for each of our customers and category values by aggregating the complete joint dataset.

```sql
DROP TABLE IF EXISTS category_rental_counts;
CREATE TEMP TABLE category_rental_counts AS --creating a Temp table
SELECT
  customer_id,
  category_name,
  COUNT(*) AS rental_count,  --aggregating on customer_id and category_name to find the count
  MAX(rental_date) AS latest_rental_date --I included the maximum rental date to use as a criteria for choosing a top category in the case there is a tie.
FROM complete_joint_dataset_with_rental_date
GROUP BY
  customer_id,
  category_name;

-- profiling just customer_id = 3 values sorted by rental_count in descending order
SELECT *
FROM category_rental_counts
WHERE customer_id = 3
ORDER BY
	customer_id,
 	rental_count DESC,
	latest_rental_date DESC
LIMIT 5;
```

**Result**
| customer_id | category_name | rental_count | latest_rental_date |
| ----------- | ----------- | ----------- | ----------- |
| 3 | Action | 4 | 2005-07-29 11:07:04.000 |
| 3 | Sci-Fi | 3 | 2005-08-22 09:37:27.000 |
| 3 | Sci-Fi | 3 | 2005-08-18 14:49:55.000 |
| 3 | Sci-Fi | 2 | 2005-08-23 07:10:14.000 |
| 3 | Sci-Fi | 2 | 2005-08-20 06:14:12.000 |

We see that for the customer with id = 3, there is a tie for the 2nd top category position. To address this, I added the max (rental_date) so that I can pick the rental with the most recent date.


2. `average_comparison`, `percentile`, `category_percentage`: 

To calculate these three data points, we need to find 3 things: 
- The average rental count for each category - (to find `average_comparison`)
- The total number of movies rented by each customer regardless of the film category - (to find `category_percentage`)
- A count of films rented by a customer for each category - (to find `percentile`). We have already found this in the previous step (`category_rental_count`)

Let’s begin:
`average_comparison`:

```sql	
DROP TABLE IF EXISTS average_category_rental_counts;
CREATE TEMP TABLE average_category_rental_counts AS
SELECT
  category_name,
  FLOOR(AVG(rental_count)) AS avg_rental_count -- To remove decimals and round down values. 
FROM category_rental_counts
GROUP BY
  category_name;

-- output the entire table by desc avg_rental_count
SELECT *
FROM average_category_rental_counts
ORDER BY
  avg_rental_count DESC
LIMIT 3;
```

| category_name | avg_rental_count |
| ----------- | ----------- |
| Action | 2 |
| Animation | 2 |
| Children | 1 |

`category_percentage`: Calculating this involves us dividing a customer’s category count by the total number of movies they have watched across all categories. We would be querying for the total number of movies rented by each customer. I would be calculating the category_percentage in a later step.

```sql
DROP TABLE IF EXISTS customer_total_rentals;
CREATE TEMP TABLE customer_total_rentals AS
SELECT
  customer_id,
  SUM(rental_count) AS total_rental_count
FROM category_rental_counts
GROUP BY customer_id;

-- show output for first 5 customer_id values
SELECT *
FROM customer_total_rentals
WHERE customer_id <= 3
ORDER BY customer_id
LIMIT 3;	
```

| customer_id | total_rental_count |
| ----------- | ----------- |
| 1 | 32 |
| 2 | 27 |
| 3 | 26 |

		     
`percentile`: The percentile simply describes how the customer ranks in terms of the top X% compared to all other customers in a film category. To do this I employed the Percent_Rank window function on the previously derived temporary table, `category_rental_count`.		     

```sql

DROP TABLE IF EXISTS customer_category_percentiles;
CREATE TEMP TABLE customer_category_percentiles AS
SELECT
  customer_id,
  category_name,
  -- I used the ceiling function to round up to nearest integer after multiplying by 100 to show percentage values.
  CEILING(
    100 * PERCENT_RANK() OVER (
      PARTITION BY category_name
      ORDER BY rental_count DESC
    )
  ) AS percentile
FROM category_rental_counts;

-- Sample
SELECT *
FROM customer_category_percentiles
WHERE customer_id = 1
ORDER BY customer_id, percentile
LIMIT 2;
		     
```
		     
| customer_id | category_name | percentile |
| ----------- | ----------- | ----------- |
| 1 | Classic | 1 |
| 1 | Comedy | 1 |


### Joning It All Together		     
In the code below, I generated calculated columns of the actual data points we need - percentile, average comparison, and category percentage.

Right now we have generated four tables which I am going to join before I pick out just the top two categories for all customers. Since I needed to keep all of the rental_count records, I used the `category_rental_count` temp table as the starting base table.
		     
```sql
DROP TABLE IF EXISTS customer_category_joint_table;
CREATE TEMP TABLE customer_category_joint_table AS
SELECT
  t1.customer_id,
  t1.category_name,
  t1.rental_count,
  t1.latest_rental_date,
  t2.total_rental_count,
  t3.avg_rental_count,
  t4.percentile,
  t1.rental_count - t3.avg_rental_count AS average_comparison,
  -- round to nearest integer for percentage after multiplying by 100
  ROUND(100 * t1.rental_count / t2.total_rental_count) AS category_percentage
FROM category_rental_counts AS t1
INNER JOIN customer_total_rentals AS t2
  ON t1.customer_id = t2.customer_id
INNER JOIN average_category_rental_counts AS t3
  ON t1.category_name = t3.category_name
INNER JOIN customer_category_percentiles AS t4
  ON t1.customer_id = t4.customer_id
  AND t1.category_name = t4.category_name;

-- inspect customer_id = 1 top 5 rows sorted by percentile
SELECT *
FROM customer_category_joint_table
WHERE customer_id = 1
ORDER BY percentile
limit 5;
		     
```		     
		     
### Singling Out The Top 2 Categories (`top_categories`)
We now have all various calculations and categories for every customer, but looking at the email template, I need to single out just the top 2 categories for each customer based on the rental count. To do this I made use of the Row_number window function together with the OVER clause. The row_number function creates a count column starting from 1 and ending at the last row in a specified window or group, in this case the window is the customer_id. Once the count gets to the next customer_id, it restarts and begins from 1. 

Here is the SQL code I wrote to pick the top 2 categories based on rental count for each customer.
		     
```sql
		     
DROP TABLE IF EXISTS top_categories;
CREATE TEMP TABLE top_categories AS (
WITH ordered_customer_category_joint_table AS (
  SELECT
    customer_id,
    ROW_NUMBER() OVER (
      PARTITION BY customer_id --The count restarts for a new customer_id
      ORDER BY rental_count DESC, -- ordering by rental count so the highest value is counted as 1 and so on
				latest_rental_date DESC -- to account for cases where there are ties in the rental_count. The category with the latest rental data would be selected instead
    ) AS category_ranking,
    category_name,
    rental_count,
    average_comparison,
    percentile,
    category_percentage
  FROM customer_category_joint_table
)
-- filter out top 2 rows from the CTE for final output
SELECT *
FROM ordered_customer_category_joint_table
WHERE category_ranking <= 2
);

SELECT *
FROM top_categories
WHERE customer_id = 1
	OR customer_id = 2
	OR customer_id = 3;		     
	
```		     

Let’s look at customers with id 1, 2, 3:
	
| customer_id | category_ranking | category_name | rental_count | average_comparison | percentile | category_percentage |
| ----------- | ----------- | ----------- | ----------- | ----------- | ----------- | ----------- |
| 1 | 1 | Classics | 6 | 4 | 1 | 19 |
| 1 | 2 | Comedy | 5 | 4 | 1 | 16 |
| 2 | 1 | Sports | 5 | 3 | 3 | 19 |
| 2 | 2 | Classics | 4 | 2 | 2 | 15 |
| 3 | 1 | Action | 4 | 2 | 5 | 15 |
| 3 | 2 | Sci-Fi | 3 | 1 | 15 | 12 |	
		     
Comparing the table above and the email template from the marketing team, we can see that we have successfully found the data points for top 2 categories and the category insights which are our requirements 1, 2 and 3.
	
![top 2 categories](https://raw.githubusercontent.com/CODEORDIETRYING/Marketing-Analytics-Case-Study/main/Images/Top%202%20categories%20and%20Insights.PNG)	

The top_categories table generated above contains insights of the top2 categories for each customers. To make things easier to identify, I am going to break it down to two tables: first_category_insights and second_category_insights.
	
### First Category Insight

```sql
	
DROP TABLE IF EXISTS first_category_insights;
CREATE TEMP TABLE first_category_insights AS
SELECT
  base.customer_id,
  base.category_name,
  base.rental_count,
  base.rental_count - average.category_average AS average_comparison,
  base.percentile
FROM top_category_percentile AS base
LEFT JOIN average_category_count AS average
  ON base.category_name = average.category_name;	

```	
	
### Second Category Insight

```sql
	
DROP TABLE IF EXISTS second_category_insights;
CREATE TEMP TABLE second_category_insights AS
SELECT
  top_categories.customer_id,
  top_categories.category_name,
  top_categories.rental_count,
  -- need to cast as NUMERIC to avoid INTEGER floor division!
  ROUND(
    100 * top_categories.rental_count::NUMERIC / total_counts.total_count
  ) AS total_percentage
FROM top_categories
LEFT JOIN total_counts
  ON top_categories.customer_id = total_counts.customer_id
WHERE category_rank = 2;	

```		
		
</details>

---

<details>
<summary>Requirement 4: Category Film Recommendations</summary>
To find the category recommendations, I am going to rank films by the number of times they have been rented, and this involves generating a count of movies using film_id and title.

To find these movies, I generated a summarized film count table with the category included, then I created a previously watched films table for the top 2 categories to exclude for each customer (category_films_exclusions), and finally I performed an anti join to remove the category_films exclusion from the film_counts table and used a Rank_number windows function to find the actual category_recommendations.
	

### Film Counts	
	
```sql
	
DROP TABLE IF EXISTS second_category_insights;
CREATE TEMP TABLE second_category_insights AS
SELECT
  top_categories.customer_id,
  top_categories.category_name,
  top_categories.rental_count,
  -- need to cast as NUMERIC to avoid INTEGER floor division!
  ROUND(
    100 * top_categories.rental_count::NUMERIC / total_counts.total_count
  ) AS total_percentage
FROM top_categories
LEFT JOIN total_counts
  ON top_categories.customer_id = total_counts.customer_id
WHERE category_rank = 2;	

```	
	
### Category Film Exclusions
For this step in the recommendation, I generated a table with all customer’s previously watched films so I do not recommend them something they have watched before.	
	
```sql
	
DROP TABLE IF EXISTS category_film_exclusions;
CREATE TEMP TABLE category_film_exclusions AS
SELECT DISTINCT
  customer_id,
  film_id
FROM complete_joint_dataset;

SELECT *
FROM category_film_exclusions
LIMIT 5;	

```	
	
### Final Category Recommendations
To exclude previously watched films from the recommendation, I made use of an ANTI JOIN on the category_film_exclusions table using a WHERE NOT EXISTS clause. After the exclusion, I selected just the top three films using the DENSE_RANK windows function.
	
	
```sql
	
DROP TABLE IF EXISTS category_recommendations;
CREATE TEMP TABLE category_recommendations AS
WITH ranked_films_cte AS (
  SELECT
    top_categories.customer_id,
    top_categories.category_name,
    top_categories.category_rank,
    film_counts.film_id,
    film_counts.title,
    film_counts.rental_count,
    DENSE_RANK() OVER (
      PARTITION BY
        top_categories.customer_id,
        top_categories.category_rank
      ORDER BY
        film_counts.rental_count DESC,
        film_counts.title
    ) AS reco_rank
  FROM top_categories
  INNER JOIN film_counts
    ON top_categories.category_name = film_counts.category_name
-- Using an ANTI JOIN for the exclusion  
  WHERE NOT EXISTS (
    SELECT 1
    FROM category_film_exclusions
    WHERE
      category_film_exclusions.customer_id = top_categories.customer_id AND
      category_film_exclusions.film_id = film_counts.film_id
  )
)
SELECT * FROM ranked_films_cte
WHERE reco_rank <= 3; -- To pick the top 3 recommendations.

SELECT *
FROM category_recommendations
WHERE customer_id = 1
ORDER BY category_rank, reco_rank;	

```

| customer_id | category_name | category_rank | film_id | title | rental_count | reco_rank |
| ----------- | ----------- | ----------- | ----------- | ----------- | ----------- | ----------- |
| 1 | Classics | 1 | 891 | TIMBERLAND SKY | 31 | 1 |
| 1 | Classics | 1 | 358 | GILMORE BOILED | 28 | 2 |
| 1 | Classics | 1 | 951 | VOYAGE LEGALLY | 28 | 3 |
| 1 | Comedy | 2 | 1000 | ZORRO ARK | 31 | 1 |
| 1 | Comedy | 2 | 127 | CAT CONEHEADS | 30 | 2 |
| 1 | Comedy | 2 | 638 | OPERATION OPERATION | 27 | 3 |		    
			
</details>

---
	
<details>
<summary>Requirement 5: Favorite Actor Insight & Recommendations</summary>

### Actor Base Table
For actor insight, we only need the actors name and rental count. To begin the analysis on actors, I needed to create a new base table that included the actor tables (dvd_rentals.film_actor & dvd_rentals.actor). 

```sql

DROP TABLE IF EXISTS actor_joint_dataset;
CREATE TEMP TABLE actor_joint_dataset AS
SELECT
  rental.customer_id,
  rental.rental_id,
  rental.rental_date,
  film.film_id,
  film.title,
  actor.actor_id,
  actor.first_name,
  actor.last_name
FROM dvd_rentals.rental
INNER JOIN dvd_rentals.inventory
  ON rental.inventory_id = inventory.inventory_id
INNER JOIN dvd_rentals.film
  ON inventory.film_id = film.film_id
-- different to our previous base table as we know use actor tables
INNER JOIN dvd_rentals.film_actor
  ON film.film_id = film_actor.film_id
INNER JOIN dvd_rentals.actor
  ON film_actor.actor_id = actor.actor_id;

SELECT *
FROM actor_joint_dataset
LIMIT 5;
	
```
	
| customer_id | rental_id | film_id | rental_date | title | actor_id | first_name | last_name |
| ----------- | ----------- | ----------- | ----------- | ----------- | ----------- | ----------- | ----------- |
| 130 | 1 | 80 | 2005-05-24 22:53:30 | BLANKET BEVERLY | 200 | THORA | TEMPLE |
| 130 | 1 | 80 | 2005-05-24 22:53:30 | BLANKET BEVERLYD | 193 | BURT | TEMPLE |
| 130 | 1 | 80 | 2005-05-24 22:53:30 | BLANKET BEVERLY | 173 | ALAN | DREYFUSS |
| 130 | 1 | 80 | 2005-05-24 22:53:30 | BLANKET BEVERLY | 16 | FRED | COSTNER |
| 130 | 1 | 80 | 2005-05-24 22:54:33 | BLANKET BEVERLY | 147 | FAY | WINSLET |

	
### Favorite Actor Insight	
	
I also needed to know how many times a customer has rented a movie starring a particular actor. I generated a rental count per actor for each customer and ranked this using a Dense_rank window function. 	
	
```sql

DROP TABLE IF EXISTS top_actor_counts;
CREATE TEMP TABLE top_actor_counts AS
WITH actor_counts AS (
  SELECT
    customer_id,
    actor_id,
    first_name,
    last_name,
    COUNT(*) AS rental_count,
    -- we also generate the latest_rental_date just like our category insight
    MAX(rental_date) AS latest_rental_date
  FROM actor_joint_dataset
  GROUP BY
    customer_id,
    actor_id,
    first_name,
    last_name
),
ranked_actor_counts AS (
  SELECT
    actor_counts.*,
    DENSE_RANK() OVER (
      PARTITION BY customer_id
      ORDER BY
        rental_count DESC,
        latest_rental_date DESC,
        -- just in case we have any further ties, we'll throw in the names too
        first_name,
        last_name
    ) AS actor_rank
  FROM actor_counts
)
SELECT
  customer_id,
  actor_id,
  first_name,
  last_name,
  rental_count
FROM ranked_actor_counts
WHERE actor_rank = 1;

SELECT *
FROM top_actor_counts
LIMIT 5;
	
```	

| customer_id | actor_id | first_name | last_name | rental_count |
| ----------- | ----------- | ----------- | ----------- | ----------- |
| 1 | 37 | VAL | BOLGER | 6 |
| 2 | 107 | GINA | BOLGER | 5 |
| 3 | 150 | JAYNE | NOLTE| 4 |
| 4 | 102 | WALTER | NOLTE | 4 |
| 5 | 12 | KARL | NOLTE | 4 |
	
	
### Actor Film Counts
Just like I did for the category film recommendations, I queried for the actor film counts then found the actor film exclusion and finally performing an anti join to remove the exclusions from the count table. Here are the steps:	
	
```sql

DROP TABLE IF EXISTS actor_film_counts;
CREATE TEMP TABLE actor_film_counts AS
WITH film_counts AS (
  SELECT
    film_id,
    COUNT(DISTINCT rental_id) AS rental_count
  FROM actor_joint_dataset
  GROUP BY film_id
)
SELECT DISTINCT
  actor_joint_dataset.film_id,
  actor_joint_dataset.actor_id,
  actor_joint_dataset.title,
  film_counts.rental_count
FROM actor_joint_dataset
LEFT JOIN film_counts
  ON actor_joint_dataset.film_id = film_counts.film_id;

SELECT *
FROM actor_film_counts
LIMIT 5;
	
```	
	
| film_id | actor_id | rental_count |
| ----------- | ----------- | ----------- |
| 80 | 200 | 12 |
| 80 | 193 | 12 |
| 80 | 173 | 12 |
| 80 | 16 | 12 |
| 333 | 147 | 17 |
	

### Actor Film Exclusions
For this actor film exclusions, we need to also exclude previously recommended films as well as previously watched films, so I made use a UNION to combine previous recommendations and previously watched films to make one actor film exclusions.
	
```sql

DROP TABLE IF EXISTS actor_film_exclusions;
CREATE TEMP TABLE actor_film_exclusions AS
(
  SELECT DISTINCT
    customer_id,
    film_id
  FROM complete_joint_dataset
)
-- we use a UNION to combine the previously watched and the recommended films
UNION
(
  SELECT DISTINCT
    customer_id,
    film_id
  FROM category_recommendations
);

SELECT *
FROM actor_film_exclusions
LIMIT 5;
	
```		

| customer_id | film_id |
| ----------- | ----------- |
| 493 | 567 |
| 114 | 789 |
| 596 | 103 |
| 176 | 121 |
| 459 | 724 |
	
	
### Final Actor Recommendations
I performed a similar operation to that for the category recommendations only that I used different tables (top_actor_counts, actor_film_counts and actor_film_exclusions).	
	
```sql

DROP TABLE IF EXISTS actor_recommendations;
CREATE TEMP TABLE actor_recommendations AS
WITH ranked_actor_films_cte AS (
  SELECT
    top_actor_counts.customer_id,
    top_actor_counts.first_name,
    top_actor_counts.last_name,
    top_actor_counts.rental_count,
    actor_film_counts.title,
    actor_film_counts.film_id,
    actor_film_counts.actor_id,
    DENSE_RANK() OVER (
      PARTITION BY
        top_actor_counts.customer_id
      ORDER BY
        actor_film_counts.rental_count DESC,
        actor_film_counts.title
    ) AS reco_rank
  FROM top_actor_counts
  INNER JOIN actor_film_counts
    -- join on actor_id instead of category_name!
    ON top_actor_counts.actor_id = actor_film_counts.actor_id
  -- This is a tricky anti-join where we need to "join" on 2 different tables!
  WHERE NOT EXISTS (
    SELECT 1
    FROM actor_film_exclusions
    WHERE
      actor_film_exclusions.customer_id = top_actor_counts.customer_id AND
      actor_film_exclusions.film_id = actor_film_counts.film_id
  )
)
SELECT * FROM ranked_actor_films_cte
WHERE reco_rank <= 3;

SELECT *
FROM actor_recommendations
ORDER BY customer_id, reco_rank
LIMIT 6;
	
```	
	
| customer_id | first_name | last_name | rental_count | title | film_id | actor_id | reco_rank |
| ----------- | ----------- | ----------- | ----------- | ----------- | ----------- | ----------- | ----------- |
| 1 | VAL | BOLGER | 6 | PRIMARY GLASS | 697 | 3 | 1 |
| 1 | VAL | BOLGER | 6 | ALASKA PHANTOM | 12 | 3 | 2 |
| 1 | VAL | BOLGER | 6 | METROPOLIS COMA | 572 | 3 | 3 |
| 1 | GINA | DEGENERES | 5 | GOODFELLAS SALUTE | 369 | 1 | 1 |
| 1 | GINA | DEGENERES | 5 | WIFE TURN | 973 | 1 | 2 |
| 1 | GINA | DEGENERES | 5 | DOGMA FAMILY | 239 | 1 | 3 |		
	   
</details>

















