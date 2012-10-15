---
layout: post
title: "Using Ruby % (Percent) Notation"
date: 2012-10-15 18:04
comments: true
categories: 
- ruby
---

If you seldom use Ruby percent (%) notation in daily work, here's a
**quick summary** of what I picked up recently.

Delimiter allows any non-alphanum
----
You can use any **non alpha-numeric character** as delimiter:
```ruby
%(any alpha-numeric)
%[char can be]
%%used as%
%!delimiter\!! # need to unescape !
```

Bracket pairs no need to escape
----
**No need to escape bracket pairs**, even when **nested**. 
If escape, you need to escape both opening and closing brackets.
```ruby
%( (pa(re(nt)he)sis) ) #=> "(pa(re(nt)he)sis)"
%[ [square bracket] ]  #=> "[square bracket]"
%{ {curly bracket} }   #=> "{curly bracket}"
%< <pointy bracket> >  #=> "<pointy bracket>"
```

Modifiers for String, Regex, Array, Symbol, Shell command
----
We often use % notation to **create String and Array** literals. But it
also supports **Symbol, Regex and shell command**.
```ruby
%(interpolated string (#{ "default" }))
  #=> "interpolated string (default)"
%Q(interpolated string (#{ "default" }))
  #=> "interpolated string (default)"
%q(non-interpolated string)
  #=> "non-interpolated string"
%r(#{ "interpolated" } regexp)i
  #=> /interpolated regexp/i
%w(non-interpolated\ string  separated\ by\ whitespaces)
  #=> ['non-interpolated string', 'separated by whitespaces']
%W(interpolated\ string #{ "separated by whitespaces" })
  #=> ['interpolated string', 'separated by whitespaces']
%s(non-interpolated symbol)
  #=> :'non-interpolated symbol'
%x(echo #{ "interpolated shell command" })
  #=> "interpolated shell command\n"
```

Anyway, I have [some minitest examples](https://github.com/teohm/a-dip-in-ruby/blob/master/spec/percent_notation_spec.rb) in case you interested.
