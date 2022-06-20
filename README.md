# Marketing Campaign Case Study 
>A marketing campaign case study that uses user data to increase email open rate and engagement for a ficticious dvd rental company called LetFlix (sounds original to me)! This case study showcases my understanding of SQL and is a part of the Serious SQL course by [Danny Ma](https://www.linkedin.com/in/datawithdanny/)

---
![Postgres](https://img.shields.io/badge/postgres-%23316192.svg?style=for-the-badge&logo=postgresql&logoColor=white)
---
## Table of Content 📑
1. 🎞 Introduction
2. 🎬 Problem
3. 🚦 Requirements
4. 🛰 Data Exploration
5. 🔬 Reverse Engineering
6. 🔗 Table Joining
7. 🔨 Problem Solving with SQL
8. 💽 Solution
---
## 🎞 Introduction
As humans, we like to feel special and not get lost in the crowd, even if an experience is a general one. Every digital marketer and business lead/owner knows how important it is to send out emails that are engaging and can get customers to take action. One tested way to engage customers is by sending personalized emails. Personalized customer emails based off user analytics are a great way for consumer-facing digital companies to connect with their audience and increase user engagement via mail. 

In this case study, I am going to be making use of SQL to leverage customer data of a fictitious movie rental company in order to send out personalized emails that engage customers and make them feel special, let’s call this company “Letflix” - sounds very original right? 
The operations in this case study can be applied elsewhere regardless of the industry and use case, but it is best suited for consumer-facing businesses. Special shout out to [Danny Ma](https://www.linkedin.com/in/datawithdanny/) for this [SQL Bootcamp](https://www.datawithdanny.com/) course. Hands down the best dollar I have ever spent learning data analytics with SQL. Let's head over to the case study problem!

---
## 🎬 Problem
The marketing team at Letflix has decided to launch their first-ever personalized marketing campaign as a strategy to engage customers and recommend new movies to them. One of the challenges they face is getting access to the necessary data points needed to populate specific parts of the email campaign. Most of the companies user data is stored within multiple tables in a relational database. To this end, they have reached out to the data team (me, myself, and I) with a draft of the email template to be sent out, in order to analyze the company’s database and come up with a single table that is much easier for them to use. 

---
## 🚦 Requirements
The marketing team has shared a draft template of the email campaign intended to be sent out to all customers. This serves as a guide for the data team. 

**IMAGE**

Let's break down the template:

**Requirement 1: Top 2 Categories**
<details>
<summary>View details</summary>
We see that the marketing team wants to let all customers know their top two most-watched film categories, hence we need to go through all customers’ rental history to determine their top two most rented film categories. The top 2 categories here are Comedy and Romance.
  **IMAGE**
</details>

**Requirement 2&3: Individual Category Insights**
<details>
<summary>View details</summary>

For the 1st category, we need to provide the following to the marketing team (Requirement 2):
  1. The total number of films watched in that category
  2. The extra number of films a customer has watched in that category, compared to the average LetFlix customer (i.e. if the average number of films rented out in the comedy category is 5 and the customer has rented out 8 comedy films, we need to display 3 to the marketing team)
  3. The customers rank in terms of the top X% compared to all other customers in this film category.

For the second top category (Requirement 3):
  1. The total number of films watched in that category
  2. The portion of each customer’s total films watched in that category compared to their total film watched across all categories.
>It should be noted that all numbers in the campaign need to be rounded up or down to avoid the presence of decimals.
  
  **IMAGE**
  
</details>

**Requirement 4: Category Film Recommendations**
<details>
<summary>View details</summary>
The marketing team also asked that we provide the top three most popular films for each of the customers’ top 2 categories. It should be noted that we cannot recommend a film that a customer has previously watched. 
  **IMAGE**
</details>

**Requirement 5: Category Film Recommendations**
<details>
<summary>View details</summary>
In addition to the top 2 categories, their insights, and recommendations, marketing has also requested for each customer’s top actor and recommended films starring that actor. As well as the count of films watched where the actor was a cast.

Note: In the case where there is a tie in top actors, the marketing team has concluded that we can order the actors in alphabetical order. Also, we are not including a recommendation that has been previously watched or recommended in the top 2 categories. If the customer doesn’t have at least 1 film recommendation - they also need to be flagged with a separate actor exclusion flag. 
  **IMAGE**
</details>

In summary, we see that we need to answer the following questions from the dataset to arrive at the results we need:

1. Identify the top 2 categories for each customer based on a user’s past rental history.
2. For each customer recommend up to 3 popular unwatched films for each category.
3. Generate 1st category insights that include:
    - How many total films have they watched in their top category?
    - How many more films have the customer watched compared to the average LetFlix customer?
    - How does the customer rank in terms of the top X% compared to all other customers in this film category?
4. Generate 2nd insights that include:
    - How many total films has the customer watched in this category?
    - What proportion of each customer’s total films watched does this count make?
5. Identify each customer’s favorite actor and film count, then recommend up to 3 more unwatched films starring the same actor.

---
##  🛰 Data Exploration
The data needed for analysis are stored in 7 different tables. To make the dataset easy to visualize and link all foreign keys to one another, each table and respective columns have been properly visualized in an entity-relationship diagram (ERD) using [dbdiagram.io.] (https://dbdiagram.io)

**VIEW DATA EXPLORATION FOLDER**

---
## 5. 🔬 Reverse Engineering
To determine what tables and columns we are going to be working with, we need to start from the end result (email template) and walk our way to the needed tools (in this case data fields) we are going to be working with. Our overall end result for this analysis is being able to populate the email template as shown by the marketing team, so we are going to be using that. 

**VIEW RESERVE ENGINEERING FOLDER**

--- 
## 6. 🔗 Table Joining
The rental table determines a lot in this analysis because it gives us information on each film rental which is what we want to analyze, but the rental table only shows us unique rental (rental_id), customer information (customer_id) and item rented (inventory_id). We do not know anything about the film in particular that was rented out as well as the film category. To know the category of the rented movie, we need to find a way to join the rental table to the category table. 

**We can do this by following these steps:**

- Step 1: Starting with the dvd_rental.Rental table, use the foreign key (inventory_id) to join it to the inventory table
- Step 2: Join the new table from step 1 to dvd_rental.film to get the film titles. The foreign key for the join is the film_id column
- Step 3: Join the dvd_rental.film table to the dvd_rental.film_category table on the film_id column
- Step 4: Finally join the previous table to the dvd_rental.category table on the category_id column serving as a foreign key

---
## 7. 🔨 Problem Solving with SQL

**Joint Dataset 👇**
<details>
<summary>Base Table (complete_joint_dataset)</summary>

```
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

| customer_id | film_id | title | rental_date | category_name |
| ----------- | ----------- | ----------- | ----------- | ----------- |
| 130 | 80 | BLANKET BEVERLY | 2005-05-24 22:53:30 | Family |
| 459 | 333 | FREAKY POCUS | 2005-05-24 22:54:33 | Music |
| 408 | 373 | GRADUATE LORD | 2005-05-24 23:03:39 | Children |
| 333 | 535 | LOVE SUICIDES | 2005-05-24 23:04:41 | Horror |
| 222 | 450 | IDOLS SNATCHERS | 2005-05-24 23:05:21 | Children |
</details>

---
## 8. 💽 Solution
To simplify things for the marketing team, I generated a single lookup table for them to consume. Each row/data entry in the lookup table is associated with a single customer.

I made use of multiple CTEs to perform the transformation from long to wide. Also, I used the CONCAT function to concatenate strings and column inputs.

**VIEW SOLUTION FOLDER**














