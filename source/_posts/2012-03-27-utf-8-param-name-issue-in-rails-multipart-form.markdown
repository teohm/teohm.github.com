---
layout: post
title: "UTF-8 param name issue in Rails multipart form"
date: 2012-03-27 21:36
comments: true
categories: 
- rails
---

I first stumbled upon this issue when [Yasith](http://thekindof.me) 
([@meaningful](https://twitter.com/meaningful)) showed me a strange bug 
in a Rails project. Here's what happened:
 
## Issue
When submit a multipart form that contains **Unicode parameter name** e.g.

```html
<form method="post" enctype="multipart/form-data" action="">
  <input name="Iñtërnâtiônàlizætiøn_name" 
         value="Iñtërnâtiônàlizætiøn_value" />
</form>
```

Rails controller returns the param value `"Iñtërnâtiônàlizætiøn_value"` as expected. 

But the **param name becomes**:
`"I\xC3\xB1t\xC3\xABrn\xC3\xA2ti\xC3\xB4n\xC3\xA0liz\xC3\xA6ti\xC3\xB8n_name"`.

It makes life miserable, if you are not expecting this to happen:
```ruby
params["Iñtërnâtiônàlizætiøn_name"] # => nil
params["I\xC3\xB1t\xC3\xABrn\xC3\xA2ti\xC3\xB4n\xC3\xA0liz\xC3\xA6ti\xC3\xB8n_name"] # => "Iñtërnâtiônàlizætiøn_value"
```

## What happened?

When [**Rack**](https://github.com/rack/rack/blob/master/lib/rack/multipart/parser.rb) 
returns multipart form data to Rails, it returns:
```ruby
{ "I\xC3\xB1t\xC3\xABrn\xC3\xA2ti\xC3\xB4n\xC3\xA0liz\xC3\xA6ti\xC3\xB8n_name" =>
  "I\xC3\xB1t\xC3\xABrn\xC3\xA2ti\xC3\xB4n\xC3\xA0liz\xC3\xA6ti\xC3\xB8n_value" } 
```

However, `ActionDispatch::Http::Parameters#encode_params` in [**Rails**](https://github.com/rails/rails/blob/master/actionpack/lib/action_dispatch/http/parameters.rb)
decided to ** *only* encode parameter values**, but *not* parameter names. **As a result, we get:**
```ruby
{ "I\xC3\xB1t\xC3\xABrn\xC3\xA2ti\xC3\xB4n\xC3\xA0liz\xC3\xA6ti\xC3\xB8n_name" =>
  "Iñtërnâtiônàlizætiøn_value" } 
```

## Solutions?

1. **Don't use Unicode param name**.
2. Patch Rails source code. I added a [fix in my forked branch](https://github.com/teohm/rails/compare/multipart_utf8_param), and [reported the issue](https://github.com/rails/rails/issues/5606). Hopefully it will get fixed soon in the coming release.

