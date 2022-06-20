If we look at the template we easily see two major sections, the category data points and 
the actor data points, there are two categories and one favorite actor required. With the 
requirements outlined, let’s look at what our SQL output would look like, we see that there 
are 15 data points needed for each customer to populate the email template:

**Category Data Points**
1. 1st ranking category name: ***cat_1***
2. 1st ranking category customer insight: ***insight_cat_1***
3. 1st ranking category film recommendations: ***cat_1_reco_1, cat_1_reco_2, cat_1_reco_3***
4. 2nd ranking category name: ***cat_2***
5. 2nd ranking category customer insight: ***insight_cat_2***
6. 2nd ranking category film recommendations: ***cat_2_reco_1, cat_2_reco_2, cat_2_reco_3***

**Actor Data Points**
1. Top actor name: ***actor***
2. top actor insight: ***insight_actor***
3. Actor film recommendations: ***actor_reco_1, actor_reco_2, actor_reco_3***

The final SQL output table should contain a single row for each customer_id and all 15 data points 
as columns. 

Some of these 15 data points are not readily available from just scanning through the dataset. 
Only the category and actor names are directly evident. To get to our other data points, 
we would need to perform some joins and calculations. Let’s re-arrange the sections so that 
we make more sense of the data points and how to arrive at them.

---

## **1. Top Categories and Actor Insights [insight_cat_1, insight_cat_2 and insight_actor]**

![Email Template](https://raw.githubusercontent.com/CODEORDIETRYING/Marketing-Analytics-Case-Study/main/Images/LetFlix%20DVD%20Rental%20Company%20Marketing%20Case%20Study.png)

Let’s take a look at the top categories and actor insights and highlight the necessary data points:

**1st Category Insights (insight_cat_1):**

*You’ve watched {`rental_count`} {`category_name`} films, that’s {`average_comparison`} more than the LetFlix average and puts you in the top {`percentile`}% of {`category_name`} gurus!*

**2nd Category Insights (insight_cat_2):**

*You’ve watched {`rental_count`} {`category_name`} films making up {`category_percentage`}% of your entire viewing history!*

**Actor Insights (insight_actor):**

You’ve watched <`rental_count`> films featuring <`actor_name`>! Here are some other films <`first_name`> stars in that might interest you!

Hence, to generate each customer insight as seen from the email template, we need the following inputs:

- `category_name`: The name of the top or second-ranking categories
- `rental_count`: There are three rental counts here the first two are related to the count of movies rented in the top categories, while the actor rental count is a count of the number of films rented that feature the customer’s favorite actor.
- `average_comparison`: How many more films has the customer watched compared to the average LetFlix customer? For this we would need to count the number of times a customer has rented a film in their top category and subtract the number from the average count or rentals across that category. Hence, if a customers top category is
- `percentile`: How does the customer rank in terms of the top X% compared to all other customers in this film category?
- `category_percentage`: What proportion of each customer’s total films watched does this count make?
- `actor_name` : The full name of the customers’ top actor
- `top_actor_counts`: The number of times a customer has rented a movie featuring their top actor.

---

## 2. Film Category and Actor Recommendations [***cat_1_reco_1/2/3, cat_2_reco_1/2/3, actor_reco_1/2/3***]

This category represents a customer’s individual film recommendations for each category and top actor. 
The film recommendations are based on overall popularity across all customers. It should be noted that the 
customers should not have seen the movies before and the movie should not be recommended more than once across 
a category or actor-based recommendations. 

---

## **Conclusion**

On looking at the given column labels and the data points we need to generate, we clearly see that no 
table is sufficient in itself to give us all the information we need, hence we need to perform multiple 
join operations.

![ERD](https://raw.githubusercontent.com/CODEORDIETRYING/Marketing-Analytics-Case-Study/main/Images/ERD%20-%20LetFlix%20Reverse%20Engineering.png)

If we look at the top 2 categories and top actors - their insights and recommendations, we see that we need data from all tables:

- Rental: To know each customer’s id and individual rental
- Film: To get the title of movies for recommendation
- Inventory: Since this table contains the inventory_id and film_id, it acts as a bridge that links the rental and film table together
- Category: So that we can locate the appropriate category names, including the top 2 categories
- Film_category: Since this table contains the film_id and category_id, it act as a bridge that links the film and category tables together
- Actor: To get the actor full names
- Film_actor: Since this table contains the film_id and actor_id, it acts as a bridge that links the actor table to the film_category and film tables. This allows us to get film recommendations from a customer’s top actor.

It should be noted that we do not need the film_actor and actor tables when finding the data points for the top 2 categories. The top 2 categories are only interested in “categories” for each customer, we are not particular about who acts in these films. When computing the data points for the top actor, actor insight and film recommendations, then we would need the film_actor and actor tables.

Now we know that we need to conduct multiple joins and a few calculations to arrive at the needed data points for the marketing team.






