---
layout: post
title: ActiveRecord Model Equality
description: >
  In Rails ActiveRecord, how to check if 2 model objects are equal, in a
  proper way. 
comments: true
keywords: rails, ruby, active-record, model, equality, howto.
categories: 
- rails
- rails-model
---

In daily web app development, we often need to check if two model objects are equal.
When I first started using Rails, this is what I wrote:

{% highlight ruby %}
if user.id == current_user.id
  # do something
end
{% endhighlight %}

Use obj1 == obj2 instead
------
Of course, that reflects my ignorance on learning the new tool.

So, here's a proper way to compare if 
[two model objects are equal](http://api.rubyonrails.org/classes/ActiveRecord/Base.html#method-i-3D-3D):
{% highlight ruby %}
test "2 records with same #id equal to each others." do
  c1 = Comment.create
  c2 = Comment.find(c1.id)

  assert c1 == c2
  assert c2 == c1
end
{% endhighlight %}

What about new record?
----
Basically, a new model object only equals to itself:
{% highlight ruby %}
test "New record does not equal to another new record." do
  new_record = Comment.new
  another_new = Comment.new

  assert new_record == new_record
  assert another_new == another_new
  assert new_record != another_new
end
{% endhighlight %}

Learning to use `==` operator for checking model equality
is useful. Because now, we can start using other methods
that depend on `==` operator. For instance, `Array#include?`.

{% highlight ruby %}
test "When applied in Array" do
  c1 = Comment.create
  c2 = Comment.find(c1.id)

  assert [c1].include? c2
  assert [c2].include? c1
  assert [c1, c2] == [c1, c2]
end
{% endhighlight %}

Rails learning tests
---
You may notice the code examples are written as tests. 
This is my first [Rails learning test](https://github.com/teohm/lt_rails).
So the plan is to write more tests as a way to learn Rails. I will
post them here occationally if interesting enough to share.

Source code: [model_equality_test.rb](https://github.com/teohm/lt_rails/blob/master/test/unit/model_equality_test.rb)

