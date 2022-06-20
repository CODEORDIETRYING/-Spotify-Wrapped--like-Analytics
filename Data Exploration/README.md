All the company's user data is stored in an Entity Relationship Diagram (ERD)
![ERD Diagram](https://raw.githubusercontent.com/CODEORDIETRYING/Marketing-Analytics-Case-Study/main/Images/ERD%20-%20LetFlix.png)

<details>
  <summary>Here is the code for creating the ERD seen above</summary>
  
  ```
  
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

