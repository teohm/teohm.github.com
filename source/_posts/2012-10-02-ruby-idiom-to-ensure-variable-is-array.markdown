---
layout: post
title: "Ruby idiom to ensure variable is Array"
date: 2012-10-02 23:27
comments: true
categories: 
- ruby
---

**Stop** doing this:
```ruby
arry = input || []  # handle input == nil
arry = [input] unless arry.kind_of?(Array)  # handle single value object

arry.each do |item|
  #process item
end
```

In Ruby, you should use
[Kernel#Array](http://www.ruby-doc.org/core-1.9.3/Kernel.html#method-i-Array)
to **convert the variable into an array object**:

```ruby
Array(input).each do |item|
  # process item
end

# Array(nil)     # => []
# Array("foo")   # => ["foo"]
# Array([1,2,3]) # => [1,2,3]
```

[More usage examples](https://github.com/teohm/a-dip-in-ruby/blob/master/spec/kernel_array_spec.rb) written in minitest.

