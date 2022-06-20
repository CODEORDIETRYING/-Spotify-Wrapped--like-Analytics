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
  
</details>




















