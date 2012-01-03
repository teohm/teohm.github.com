---
layout: post
title: ActiveRecord Mass Assignment
comments: true
categories: [rails-model, lt-rails]
---

Even you haven't heard of 
[mass assignment](http://guides.rubyonrails.org/security.html#mass-assignment), 
it already exists in your first Rails generated scaffold code.

{% highlight ruby %}
def create
  @comment = Comment.new(params[:comment])
  ..
end

def update
  ..
  @comment.update_attributes(params[:comment])
  ..
end
{% endhighlight %}

Why knowing mass assignment is important?
----
By default, mass assignment opens up an undesirable **security hole**, 
by allowing web clients to update any attributes they passing in, 
including attributes like `created_at`.

For details, you can read:

- [Ruby On Rails Security Guide - Mass Assignment](http://guides.rubyonrails.org/security.html#mass-assignment)
- [Rail Spikes: Is your Rails application safe?](http://railspikes.com/2008/9/22/is-your-rails-application-safe-from-mass-assignment)

Mass assignment methods
---
There are a few **ActiveRecord methods** that accept mass assignment:

- [`new`](http://api.rubyonrails.org/classes/ActiveRecord/Base.html#method-c-new)
- [`create`](http://api.rubyonrails.org/classes/ActiveRecord/Base.html#method-c-create)
- [`attributes=`](http://api.rubyonrails.org/classes/ActiveRecord/Base.html#method-i-attributes-3D)
- [`update_attributes`](http://apidock.com/rails/ActiveRecord/Base/update_attributes)

Using any of these methods, means you are now responsible for the safety
of the web application.

Minimal protection
---
To use mass assignment safely, we want to specify exactly which attributes
allowed to be updated.

### 1. Define `attr_accessible` on every model
`attr_accessible` defines a white-list of attributes 
that can be updated during mass assignment.

{% highlight ruby %}
class Comment < ActiveRecord::Base
  attr_accessible :title, :content
end
{% endhighlight %}

### 2. Enforce `attr_accessible` usage 
You may set the white-list default to empty. This forces you
to define the whitelist explicitly on a model, before 
mass assignment can be used.

{% highlight ruby %}
# config/initializer/enforce_attr_accessible.rb

ActiveRecord::Base.send(:attr_accessible, nil)
{% endhighlight %}


