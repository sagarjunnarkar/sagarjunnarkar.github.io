---
layout:     post
title:      ActiveRecord scope returns all records when result of the scope is nil
date:       2019-09-15
summary:    Scopes are intended to return an ActiveRecord::Relation object which is composable with other scopes. It means method should be chainable. It should not return nil or false otherwise it will return all records.
categories: blogs
---

ActiveRecord scopes are used very oftenly in Rails applications. It adds a class method for retrieving and querying objects.

There is one <span style="color:red">**unusual**</span> but <span style="color:green">**expected**</span> behaviour of scope. Let's check it by example.

Suppose we need to find recently published post then scope will look like as below -

```ruby
class Post < ApplicationRecord
  scope :recent_published, -> { where(published: true).order('published_at DESC').first }
end
```

And we have only unpublished posts in table. Let's check in `rails console` -

```ruby
2.6.0 :001 > Post.recent_published
  Post Load (0.3ms)  SELECT  "posts".* FROM "posts" WHERE "posts"."published" = ? ORDER BY published_at DESC LIMIT ?  [["published", 1], ["LIMIT", 1]]
  Post Load (0.1ms)  SELECT  "posts".* FROM "posts" LIMIT ?  [["LIMIT", 11]]
 => #<ActiveRecord::Relation [#<Post id: 1, published: false, published_at: "2019-09-14 07:32:24", created_at: "2019-09-14 07:32:24", updated_at: "2019-09-14 07:32:24">, #<Post id: 2, published: false, published_at: "2019-09-14 07:32:27", created_at: "2019-09-14 07:32:27", updated_at: "2019-09-14 07:32:27">]>
```

### <span style="color:orange">Gotchaaaa!!!</span>

So you must be expecting [] but you got some results.
Let's look at the query -

```sql
Post Load (0.3ms)  SELECT  "posts".* FROM "posts" WHERE "posts"."published" = ? ORDER BY published_at DESC LIMIT ?  [["published", 1], ["LIMIT", 1]]
Post Load (0.1ms)  SELECT  "posts".* FROM "posts" LIMIT ?  [["LIMIT", 11]]
```

The first query returns nil and hence second query is executed which loads all data from posts table.

But why is this happening at all?

To find answer let's look into activerecord code. The [comment](https://github.com/rails/rails/blob/v6.0.0/activerecord/lib/active_record/scoping/named.rb#L71){:target="_blank"} about scope in ActiveRecord says -

```ruby        
# If it returns +nil+ or +false+, an
# {all}[rdoc-ref:Scoping::Named::ClassMethods#all] scope is returned instead.
```

This is very important to understand that scope should not return `nil` or `false`
otherwise it returns all records.


In our case, scope `recent_published` i.e. `Post.where(published: true).order('published_at DESC'].first` returns nil and hence second query `SELECT  "posts".* FROM "posts"` executed which load all data.


Let's try to understand the implementation of `scope` method. Let's see the method [_exec_scope#L403](https://github.com/rails/rails/blob/v6.0.0/activerecord/lib/active_record/relation.rb#L403){:target="_blank"} -

```ruby
instance_exec(*args, &block) || self
```

Below is output of `instance_exec(*args, &block)` -

```ruby
[1] pry(#<Post::ActiveRecord_Relation>)> instance_exec(*args, &block)
  Post Load (0.4ms)  SELECT "posts".* FROM "posts" WHERE "posts"."published" = ? ORDER BY published_at DESC LIMIT ?  [["published", 1], ["LIMIT", 1]]
=> nil
```

This returns nil and hence `self` is executed which returns all records -

```ruby
[2] pry(#<Post::ActiveRecord_Relation>)> self
  Post Load (0.2ms)  SELECT "posts".* FROM "posts"
```

## So what is the solution?

If you notice in `recent_published` scope, we have used `first` which is actully a problem.
This `first` method is from module ActiveRecord::FinderMethods.

We can use `limit` instead of `first`.

```ruby
scope :recent_published, -> { where(published: true).order('published_at DESC').limit(1) }
```

## How/why `limit` works and `first` didn't?

#### Short answer
`limit` returns ActiveRecord::Relation and `first` returns the first item or nil.

#### Long answer

Check how ruby works for `||` condition with `nil` vs `[]` -

```ruby
nil || anything => anything #case of using `first` in scope
[]  || anything => []       #case of using `limit` in scope
```

This is same implementations of `_exec_scope` which is `instance_exec(*args, &block) || self`.

To check further we should see the implementation of `first` method in [active record](https://github.com/rails/rails/blob/v6.0.0/activerecord/lib/active_record/relation/finder_methods.rb#L116){:target="_blank"}.

The `first` method calls [`find_nth`](https://github.com/rails/rails/blob/v6.0.0/activerecord/lib/active_record/relation/finder_methods.rb#L504){:target="_blank"}

```ruby
def find_nth(index)
  @offsets[offset_index + index] ||= find_nth_with_limit(index, 1).first
end
```
In above `find_nth_with_limit(index, 1)` which returns empty Array.

And `find_nth_with_limit(index, 1).first` is same as `[].first` which returns nil.

In case of `limit` [method](https://github.com/rails/rails/blob/v6.0.0/activerecord/lib/active_record/relation/query_methods.rb#L719){:target="_blank"}, the return object is ActiveRecord::Relation and it is chainable too.

## Summary

This is very important to note that the scope is intended to return an [ActiveRecord::Relation](https://api.rubyonrails.org/classes/ActiveRecord/Relation.html){:target="_blank"} object which is composable with other scopes. In short scopes should be chainable.

Use class method instead of scope when return object can be `nil`, `false` or **non** `#<ActiveRecord::Relation []>` like `Array`, `Hash` etc.

## Reference

<https://github.com/rails/rails/issues/21882>

---
