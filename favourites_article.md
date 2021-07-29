## Introduction

[What do we want to build]

In this post we user Rails and SQL but you can apply the same concepts to any framework.

## Structuring the postgresql database using Rails Active Record  

A **pet** can be favourited by many **users**.

A **user** can favourite many **pets**.

In this case a [has-many :through](Insert Link here) association can be set up. As per the rails documentation, this association indicates that the declaring model (i.e. the user) can be matched with zero or more instances of another model (i.e. the pet) by proceeding through a third model (i.e. the favourites).

[Insert the database flow here]

The corresponding model may look like this:
[Insert model code]

The corresponding migration may look like this:
[Insert migration code]

This will allow us to do something like this:
```
User.favourites
```
[Insert the result here]
and
```
@pet = Pet.first
@pet.favourites
```
[Insert the result here]

## Data

Now that the data is set up, we want to show a list of pets with their attributes and whether or not they were favourited by a user.

An example of what we expect the rendered JSON to look like:

[Insert image here with annotations]

There are two attributes that will need to be added to the `pet` data:
- `favourited` which indicates whether the pet was favourited by the current user.
- `total_favourites` which provide an indication of how many  times the pet was favourited across all users.

In order to determine whether a pet was favourited by the current user, we will need to check if the `favourites` table contains an entry with the `current_user` and the associated `pet_id`.

This would look like: 
