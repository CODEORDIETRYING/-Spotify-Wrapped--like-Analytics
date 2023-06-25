All the company's user data can be visualized in an Entity Relationship Diagram (ERD)

![ERD Diagram](https://raw.githubusercontent.com/CODEORDIETRYING/Marketing-Analytics-Case-Study/main/Images/ERD%20-%20LetFlix.png)

<details>
  <summary>Here is the code for creating the ERD seen above</summary>
  
  ```sql
  
  Table "rental" {
  "rental_id" integer [not null]
  "rental_date" timestamp [not null]
  "inventory_id" integer [not null]
  "customer_id" smallint [not null]
  "return_date" timestamp
  "staff_id" smallint [not null]
  "last_update" timestamp [not null, default: `now()`]

Indexes {
  inventory_id [type: btree, name: "idx_fk_inventory_id"]
  (rental_date, inventory_id, customer_id) [type: btree, unique, name: "idx_unq_rental_rental_date_inventory_id_customer_id"]
}
}

Table "inventory" {
  "inventory_id" integer [not null, default: `nextval('inventory_inventory_id_seq'::regclass)`]
  "film_id" smallint [not null]
  "store_id" smallint [not null]
  "last_update" timestamp [not null, default: `now()`]

Indexes {
  (store_id, film_id) [type: btree, name: "idx_store_id_film_id"]
}
}


Table "film" {
  "film_id" integer [not null, default: `nextval('film_film_id_seq'::regclass)`]
  "title" "character varying(255)" [not null]
  "description" text
  "release_year" year
  "language_id" smallint [not null]
  "original_language_id" smallint
  "rental_duration" smallint [not null, default: 3]
  "rental_rate" "numeric(4, 2)" [not null, default: 4.99]
  "length" smallint
  "replacement_cost" "numeric(5, 2)" [not null, default: 19.99]
  "rating" "character varying(5)" [default: "G"]
  "last_update" timestamp [not null, default: `now()`]
  "special_features" text
  "fulltext" tsvector [not null]

Indexes {
  fulltext [type: btree, name: "film_fulltext_idx"]
  language_id [type: btree, name: "idx_fk_language_id"]
  original_language_id [type: btree, name: "idx_fk_original_language_id"]
  title [type: btree, name: "idx_title"]
}
}

Table "film_category" {
  "film_id" smallint [not null]
  "category_id" smallint [not null]
  "last_update" timestamp [not null, default: `now()`]
}

Table "category" {
  "category_id" integer [not null, default: `nextval('category_category_id_seq'::regclass)`]
  "name" "character varying(25)" [not null]
  "last_update" timestamp [not null, default: `now()`]
}

Table "film_actor" {
  "actor_id" smallint [not null]
  "film_id" smallint [not null]
  "last_update" timestamp [not null, default: `now()`]

Indexes {
  film_id [type: btree, name: "idx_fk_film_id"]
}
}

Table "actor" {
  "actor_id" integer [not null, default: `nextval('actor_actor_id_seq'::regclass)`]
  "first_name" varchar(45) [not null]
  "last_name" varchar(45) [not null]
  "last_update" timestamp [not null, default: `now()`]

Indexes {
  last_name [type: btree, name: "idx_actor_last_name"]
}
}

-- many to one relationship between rental & inventory
Ref: "rental"."inventory_id" > "inventory"."inventory_id"

-- many to one inventory to film
Ref: "inventory"."film_id" > "film"."film_id"

-- one to one relationship between film_category and film 
Ref: "film_category"."film_id" - "film"."film_id"

-- many to one relationship between film_category and category
Ref: "film_category"."category_id" > "category"."category_id"

-- one to many relationship between film ands film_actor
Ref: "film"."film_id" < "film_actor"."film_id"

-- many to one relationship between film_actor and actor
Ref: "film_actor"."actor_id" > "actor"."actor_id"

-- there is also an additional relationship, however we exclude it to reduce
-- any confusion!

-- many to many relationship between inventory and film_actor
-- however dbdiagram.io only lets you refer to each combination once...
-- so we only show one direction of relationship!
-- Ref: "inventory"."film_id" > "film_actor"."film_id"
  
  ```
</details>

**Description of All Tables:**
<details>
 <summary>Table #1 - Rental</summary>
This table holds information on film rentals at a customer level. Each record is a rental instance where a customer rents out a particular film. The rental_id column serves as a unique identifier for each record in the table which corresponds to an individual customer_id renting a specific item with an inventory_id. The last_update field is an internal database timestamp for when the data row was last inserted or updated. 

From the ERD, we can observe that there is a linkage between the rental table and the inventory table via the inventory_id field.
 
</details>
  
</details>

<details>
 <summary>Table #2 - Inventory</summary>
The inventory table contains information about the specific items available in each LetFlix store. It should be noted that there can be multiple inventory items for a specific film at a unique store, meaning that a particular film can be available in multiple copies, but have different inventory IDs. For instance, Spider-Man Homecoming might have 5 copies at store #1 and an additional 3 copies in store #2 - each record in this Inventory dataset will have a separate inventory_id whilst the film_id will all be the same and the store_id changes according to the store number.

From the ERD, we can observe that there is a linkage between the inventory table and the film table via the film_id column.
 
</details>

<details>
 <summary>Table #3 - Film</summary>
Whilst we can see each customer that rented a movie, the rental date, and inventory_id of the item, we know nothing about the title and category of the film itself. Information about all films in all store locations can be found in the film table. This dataset helps us uniquely identify films by title and description among other parameters. With the information provided in this table, we can identify the title of movies rented by LetFlix customers. 

We will use the film_id column to help us join the film table unto the film_actor table so that we can identify which actors appeared in each film.
 
</details>

<details>
 <summary>Table #4 - Film_Category</summary>
This table shows us the relationship between a film_id and the corresponding category_id. As you already expect, a particular category_id can be associated with multiple film_id’s (There are multiple films in the comedy category and other categories as well).
 
</details>

<details>
 <summary>Table #5 - Category</summary>
This is a table showing unique category_id’s and the associated category_name.
 
</details>

<details>
 <summary>Table #6 - Film_actor</summary>
The film_Actor shows a relationship between actors and the films they star in based off related actor_id and film_id values. In the same way a film has multiple actors, an actor can appear in multiple films, meaning we have a many-to-many relationship between both columns in the film_actor table.

To get more information about an actor’s first and last name, we would need to join the film_actor table to the actor table on the actor_id column.
 
</details>

<details>
 <summary>Table #7 - Actor</summary>
This gives us information about all actors (id, first_name and last_name). The actor_id column is a unique key that identifies all actors. 
 
</details>
  

