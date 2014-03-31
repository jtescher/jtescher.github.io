---
layout: post
title:  "How to keep  your data consistent with foreign key constraints"
date:   2014-03-31 11:00:00
categories: rails active_record foreign_key_constraints data integrity
---

We all have that co-worker (and have been that co-worker) who SSHs into a
server and runs SQL statements against live data. On staging servers this
can be a minor issue if things go wrong, but in production it can be disastrous.
At [WillCall](https://www.getwillcall.com) we often have to spend time
putting our staging data into different configurations for testing purposes
and have felt the pain of inconsistent data more than once.

Today we're going to talk about data consistency, why keeping your data
consistent is always a challenge, and what you can do about it.

Integrity issues can creep in unintentionally if you do not have a
deep understanding of how ActiveRecord callbacks and validations work and
which methods skip them entirely!

For example if you have a `Post` model and each `Post` had many `Comments`, when you 
call `post.destroy` with the `dependent: :destroy` option, it will destroy the 
associated comments as well.

```ruby
class Post < ActiveRecord::Base
  has_many :comments, dependent: :destroy
end

class Comment < ActiveRecord::Base
  belongs_to :post
end
```

This keeps your data consistent. You have no orphaned comments floating around
waiting to explode when you call `comment.post.title`.

However if someone were to mistakenly use
post.[delete](http://api.rubyonrails.org/classes/ActiveRecord/Relation.html#method-i-delete)
instead of post.[destroy](http://api.rubyonrails.org/classes/ActiveRecord/Relation.html#method-i-destroy)
the callbacks that destroy the associated comments would never be run!

The Rails guides have a list of 
[dangerous methods that skip callbacks](http://guides.rubyonrails.org/active_record_callbacks.html#skipping-callbacks)
for various reasons and should probably be avoided by ~~junior~~ most 
developers. They include:

* decrement
* decrement_counter
* delete
* delete_all
* increment
* increment_counter
* toggle
* touch
* update_column
* update_columns
* update_all
* update_counters

### Solution: Foreign key constraints.

Foreign key constraints are the database's solution to this data integrity 
problem. They are basically a way to tell the database to only allow actual 
Post IDs in the comments `post_id` column. This means if you tried to add a 
comment to a non-existent or deleted post, or delete a post that has comments, 
it would raise an `ActiveRecord::InvalidForeignKey` exception.

Rails does not have support for foreign key constraints because
[DHH](http://en.wikipedia.org/wiki/David_Heinemeier_Hansson) is not a
fan of them, but there are several gems that will add this ability for you. 
[Foreigner](https://github.com/matthuhiggins/foreigner) is the most popular, 
but I have been having good results with the more full featured 
[SchemaPlus](https://github.com/lomba/schema_plus). I'll show how to use 
both:

### Foreigner

Foreigner is a simple gem that just adds methods to create foreign keys in
mysql, postgres, and sqlite. To install Foreigner, add the following to your
Gemfile:

```ruby
gem 'foreigner', '~> 1.6.1'
```

And then install with `$ bundle install`.

Now you can use the `foreign_key` method in your migrations and the foreign
key constraint will be added for you.

```ruby
create_table :comments do |t|
  t.string :body
  t.foreign_key :posts
end
```

And that's it! You can also add `null: false` to ensure that the post_id is 
not nil.

### SchemaPlus

[SchemaPlus](https://github.com/lomba/schema_plus) is another option. 
Although being slightly less popular it adds some conveniences that foreigner 
does not like 
[column default expressions](https://github.com/lomba/schema_plus#column-defaults-expressions)
which I might talk about in another post. Another benefit is that it will 
automatically add foreign key constraints to all `t.references` and
`t.belongs_to` methods in your migrations for you. To install schema_plus add 
the following to your Gemfile:

```ruby
gem 'schema_plus', '~> 1.4.1'
```

And then install with `$ bundle install`. And you're done! When you start your 
next project this is definitely something I would try playing with. There are 
also many options you can pass that customize your foreign key constraints 
described [here](https://github.com/lomba/schema_plus#foreign-key-constraints).


### Testing Integrity

Finally adding a few tests is always a good thing, so if you want to test that 
you cannot accidentally violate your data integrity you could do something 
like this in [rspec](https://github.com/rspec/rspec).

```ruby
require 'spec_helper'

describe 'Post Integrity' do
  it 'should not allow posts to be destroyed if they have comments' do
    post = create(:post)
    create(:comment, post: post)
    expect { post.destroy! }.to raise_error(ActiveRecord::DeleteRestrictionError)
  end

  it 'should not allow posts to be deleted if they have comments' do
    post = create(:post)
    create(:comment, post: post)
    expect { post.delete }.to raise_error(ActiveRecord::InvalidForeignKey)
  end
end
```
