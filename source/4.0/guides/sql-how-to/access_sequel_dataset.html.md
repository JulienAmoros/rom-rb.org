---
chapter: SQL How To
title: Use sequel methods on ROM::Relations
---

You like ROM? But you struggle when you need to forge more complex SQL queries? Well, brace yourself! Here is a little
trick that will allow you to use Sequel methods directly on your `ROM::Relations!

First, lets take a look at what is actually a `ROM::Relation` build with `:sql`:`
```ruby
user_relation = Repositories::User.new(rom_container).users
user_relation
=> #<Relations::Users name=ROM::Relation::Name(users) dataset=#<Sequel::Mysql2::Dataset: "SELECT `users`.`id`, `users`.`name`, `users`.`email` FROM `users` ORDER BY `users`.`id`">>
```

Every ROM::Relations helper methods that you'll use will modify it, for instance:
```ruby
user_relation.by_pk(3)
=>#<Relations::Users name=ROM::Relation::Name(users) dataset=#<Sequel::Mysql2::Dataset: "SELECT `users`.`id`, `users`.`name`, `users`.`email` FROM `users` WHERE (`applications`.`id` = 3) ORDER BY `users`.`id`">>

user_relation.where(id: [1,2,3])
=>#<Relations::Users name=ROM::Relation::Name(users) dataset=#<Sequel::Mysql2::Dataset: "SELECT `users`.`id`, `users`.`name`, `users`.`email` FROM `users` WHERE (`id` IN (1, 2, 3)) ORDER BY `users`.`id`">>

user_relation.distinct.left_join(:posts)
=>#<Relations::Users name=ROM::Relation::Name(users) dataset=#<Sequel::Mysql2::Dataset: "SELECT DISTINCT `users`.`id`, `users`.`name`, `users`.`email` FROM `users` LEFT JOIN `posts` ON (`users`.`id` = `posts`.`user_id`) ORDER BY `users`.`id`">>
```

Let's imagine that you want to use sequel method that ROM has not implemented (yet), you can access directly to the 
sequel object through the `dataset` method of your relation.

Here, we consider that you have a `users_posts` table with two integer attributes `user_id` and `post_id` and that your 
database will raise an error if you try to create twice a record with the same couple of integers. You can use sequel
methods in order to add tuples to the recordings, all in one request, and an other one to ignore duplicates, and do that 
directly in your SQL request.

```ruby
users_posts_relation
=>#<Relations::UsersPosts name=ROM::Relation::Name(users_posts) dataset=#<Sequel::Mysql2::Dataset: "SELECT `users_posts`.`user_id`, `users_posts`.`post_id` FROM `users_posts`">>

users_posts_relation.dataset
=>#<Sequel::Mysql2::Dataset: "SELECT `users_posts`.`user_id`, `users_posts`.`post_id` FROM `users_posts`">

tuples = [{user_id: 12, post_id: 100}, {user_id: 12, post_id: 101}]
users_posts_relation.dataset.insert_ignore.multi_insert(tuples)
=> ["INSERT IGNORE INTO `users_posts` (`user_id`, `post_id`) VALUES (12, 100), (12, 101)"]
```

The `multi_insert` method actually executes the query and will return the actually executed query, but won't raise an
error if one of the tuple already exists in DB!
