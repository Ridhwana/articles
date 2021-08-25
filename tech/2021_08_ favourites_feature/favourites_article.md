## Introduction

Recently, I've been working on a project in my spare time that allows people to find and adopt pets from shelters within their surrounding areas.

One of the features that I've implemented is the ability for a prospective pet parent to favourite the pets that they're interested in, as well as view the total number of times that the pet was favourited.

When working on this feature I found that there were resources that showed how to query and display a user's favourites on a single page but not any that showed all the items on one page and highlighted which were favourited by the user, or that showed the number of users that favourited each item.

This is what I mean:

[Insert screenshot of index page]

In order to build this feature, we need to:
1. Set up a `favourites` table as a relationship between the `user` and the `pet`.
2. Once the relationship is set up, write SQL to display the data in a JSON format that can be queried via the API.

I'll go through each step in detail below:

## 1. Set up the `favourites` relationship

In order to implement this feature, my database needs to have two tables in place; a `user` table and a `pet` table.

[Insert a user table and pet table model]

In order to store which pets have been favourited by which users, we need to set up a relationship between the `pet` and `user` table. The relationship can be described as follows:

- A **pet** can be favourited by many **users**.
- A **user** can favourite many **pets**.

In this case a [has-many :through association](https://guides.rubyonrails.org/association_basics.html#the-has-many-through-association) can be set up between the two tables to store the favourites. As per the rails documentation, this association indicates that the declaring model (i.e. the `user`) can be matched with zero or more instances of another model (i.e. the `pet`) by proceeding through a third model (i.e. the `favourites`).

Our database flow will now look like this:

[Insert the database flow here]

And the corresponding model:

```
class Pet < ApplicationRecord
  has_many :favourites
  has_many :users, :through => :favourites
end

class Favourite < ApplicationRecord
  belongs_to :pet
  belongs_to :user
end

class User < ApplicationRecord
  has_many :favourites
  has_many :pets, :through => :favourites
end
```

And finally, the migration:

```
class CreateFavourites < ActiveRecord::Migration[6.0]
  def change

    create_table :users do |t|
      t.string :username
      t.string :email
      t.string :password_digest

      t.timestamps
    end

    create_table :pets do |t|
      t.string :pet_type
      t.datetime :dob
      t.string :gender
      t.string :image
      t.string :breed
      t.string :name

      t.timestamps
    end

    create_table :favourites do |t|
      t.belongs_to :user
      t.belongs_to :pet

      t.timestamps
    end

  end
end
```

We could have used a `has_and_belongs_to_many` association if we really wanted to. However, spending the time to set up  a `has many through` association, provides us with the opportunity to more easily add validations, callbacks and extra attributes on our `join` table (i.e. the Favourite table.

Once we have this association in place, we can access the favourite pets of a user as follows:

```
@user = User.find(13)
@user.favourites
```

The result may look like something like this:
```
#<ActiveRecord::Associations::CollectionProxy
[
  #<Favourite id: 25, user_id: 13, pet_id: 11, created_at: "2021-07-25 10:37:47", updated_at: "2021-07-25 10:37:47">,
  #<Favourite id: 26, user_id: 13, pet_id: 13, created_at: "2021-07-25 10:38:47", updated_at: "2021-07-25 10:38:47">
]>
```

We can also easily get the users that have favourited a pet:

```
@pet = Pet.first
@pet.favourites
```

The result may look like something like this:

```
#<ActiveRecord::Associations::CollectionProxy
[
  #<Favourite id: 21, user_id: 12, pet_id: 11, created_at: "2021-07-14 15:01:26", updated_at: "2021-07-14 15:01:26">,
  #<Favourite id: 25, user_id: 13, pet_id: 11, created_at: "2021-07-25 10:37:47", updated_at: "2021-07-25 10:37:47">
]>
```

## Write SQL to display the correct data

Now that we have our relations and persisted data storage in place, we want to expose the data via an API. This means putting some JSON together.

In order to do so, we need to think about what data needs to be shown on the page. We want to show a list of pets on the home page with their attributes and whether or not they were favourited by the currently signed in user.

An example of what we expect the rendered JSON to look like:

[Insert image here with annotations]

As you can see from the annotations above, we are rendering the standard `pet` data with two additional attributes:
- `favourited` which is a boolean that indicates whether the pet was favourited by the current user.
- `total_favourites` which provides an indication of how many times the pet was favourited across all users.

### A result determining whether the pet was `favourited` by the user

In order to determine whether a pet was favourited by the current user, we will need to check if the `favourites` table contains an entry with the `current_user` with `user_id = 12` and the associated `pet_id`.

The SQL looks like this:
<!-- maybe we can add a sql variable for current_user -->

```sql
SELECT pets.*, CASE WHEN favourites.user_id = 12 THEN TRUE ELSE FALSE END AS favourited
FROM pets
LEFT JOIN favourites
ON pets.id = favourites.pet_id
AND favourites.user_id = 12
```

Let's decompose the  SQL a bit:

- We want records from the `pets` table , and the matching records from the `favourites` table.  
- The matching records are determined by the ON CLAUSE. In this case we want to join if two conditions are met, i.e. the `pets.id = favourites.pet_id` and AND `favourites.user_id = 12`
[Insert a diagram https://www.w3schools.com/sql/sql_join_left.asp]
- We use a left join because we want to returns all rows from the left table (Pets), even if there are no matches in the right table (Favourites).
- Finally, let's go back to the first line to understand what we want to display in our result. We want to display all thecolumns from the pets table  `SELECT pets.*`
- We want to return a boolean - `true` or `false` indicating wheter a user is favourited. In order to do so we evaluate if the user_id on the `favourites`  


### A result with `total_favourites`

In oder to determine the total number of times that a pet was favourited, we will need to join the favourites table to the pets table by the pet id and count the number of favourites.

The SQL looks like this:
```sql
SELECT pets.*, count(favourites.*) AS total_favourites
FROM pets
LEFT JOIN favourites
ON pets.id = favourites.pet_id
GROUP BY pets.id
```

- We left join the Pet and the Favourite table on the pet id


- The GROUP BY clause combines the results for identical values in a column or expression. This clause is typically used in conjunction with aggregate functions to generate a single figure for each unique value in a column or expression   
-The preceding query returns one row with a count for each pet in the favourites table.
https://www.postgresql.org/docs/current/sql-select.html#SQL-GROUPBY
you cannot reference a field on the SELECT statement if it doesn't appear on the GROUP BY clause or without using an aggregated function.
> When GROUP BY is present, or any aggregate functions are present, it is not valid for the SELECT list expressions to refer to ungrouped columns except within aggregate functions or when the ungrouped column is functionally dependent on the grouped columns, since there would otherwise be more than one possible value to return for an ungrouped column. A functional dependency exists if the grouped columns (or a subset thereof) are the primary key of the table containing the ungrouped column.

https://dba.stackexchange.com/questions/194337/postgres-error-must-appear-in-the-group-by-clause-or-be-used-in-an-aggregate-fu


### A result with both the `total_favourites` and `favourited`

Finally, we combine these queries to show both the `favourited` and `total_favourites` together with the pet attributes in one query.


```sql
SELECT pets.id, count(favourites.*) AS total_favourites, CASE WHEN user_favourites.favourited THEN TRUE ELSE FALSE END AS favourited from pets
LEFT JOIN (
  SELECT pet_id, CASE WHEN favourites.user_id = 12 THEN TRUE ELSE FALSE END as favourited from favourites where user_id=12
) user_favourites
ON pets.id = user_favourites.pet_id
LEFT JOIN favourites
ON pets.id = favourites.pet_id
GROUP BY pets.id, user_ favourites.favourited;
```

[Insert An explanation]


### Render a JSON payload with the result in Rails

The next step is to execute this query in Rails index index action and render the result as JSON for our API endpoint.

```sql
query = <<-SQL.squish
          SELECT pets.*, count(favourites.*) AS total_favourites, CASE WHEN user_favourites.favourited THEN TRUE ELSE FALSE END AS favourited from pets
          LEFT JOIN (
            SELECT pet_id, CASE WHEN favourites.user_id = #{current_user.id} THEN TRUE ELSE FALSE END as favourited from favourites where user_id=#{current_user.id}
          ) user_favourites
          ON pets.id = user_favourites.pet_id
          LEFT JOIN favourites
          ON pets.id = favourites.pet_id
          GROUP BY pets.id, user_favourites.favourited;
        SQL
        pets = ActiveRecord::Base.connection.execute(query)

render json: { pets: pets }
```

`SQL.squish` does
`ActiveRecord::Base.connection.execute` does


And there you have it, a JSON payload to power a page that shows favourites.
