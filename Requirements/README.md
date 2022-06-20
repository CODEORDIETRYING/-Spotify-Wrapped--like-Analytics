Let’s break down the template:

**Requirement 1: Top 2 Categories**

<details>
<summary>View Details</summary>
We see that the marketing team wants to let all customers know their top two most-watched film categories, hence we need to 
go through all customers’ rental history to determine their top two most rented film categories. 
The top 2 categories here are Comedy and Romance.
  
![Top 2 Categories](https://raw.githubusercontent.com/CODEORDIETRYING/Marketing-Analytics-Case-Study/main/Images/Top%202%20categories%20circled.PNG)
</details>

**Requirement 2&3: Individual Category Insights**

<details>
<summary>View Details</summary>
For the 1st category, we need to provide the following to the marketing team (Requirement 2):

1. The total number of films watched in that category
2. The extra number of films a customer has watched in that category, compared to the average LetFlix customer (i.e. if the average number of films rented out in the comedy category is 5 and the customer has rented out 8 comedy films, we need to display 3 to the marketing team)
3. The customers rank in terms of the top X% compared to all other customers in this film category.

For the second top category (Requirement 3):

1. The total number of films watched in that category
2. The portion of each customer’s total films watched in that category compared to their total film watched across all categories.

It should be noted that all numbers in the campaign need to be rounded up or down to avoid the presence of decimals.
  
![Individual Category Insights](https://raw.githubusercontent.com/CODEORDIETRYING/Marketing-Analytics-Case-Study/main/Images/Top%202%20categories%20insights%20circled.PNG)
</details>

**Requirement 4: Category Film Recommendations**

<details>
<summary>View Details</summary>
The marketing team also asked that we provide the top three most popular films for each of the 
customers’ top 2 categories. It should be noted that we cannot recommend a film that a customer has previously watched.   

  ![Category Film Recommendations](https://raw.githubusercontent.com/CODEORDIETRYING/Marketing-Analytics-Case-Study/main/Images/top%202%20categories%20recommendations.PNG)
</details>

**Requirement 5: Favorite Actor & Recommendations**

<details>
<summary>View Details</summary>
In addition to the top 2 categories, their insights, and recommendations, marketing has also requested for each customer’s 
top actor and recommended films starring that actor. As well as the count of films watched where the actor was a cast.

**Note:** In the case where there is a tie in top actors, the marketing team has concluded that we can order the actors in alphabetical order. 
Also, we are not including a recommendation that has been previously watched or recommended in the top 2 categories. If the customer doesn’t have at least 1 film recommendation - they also need to be flagged with a separate actor exclusion flag.

![Category Film Recommendations](https://raw.githubusercontent.com/CODEORDIETRYING/Marketing-Analytics-Case-Study/main/Images/top%202%20categories%20recommendations.PNG)
</details>

---
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


