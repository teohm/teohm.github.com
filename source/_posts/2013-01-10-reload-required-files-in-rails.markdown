---
layout: post
title: "Reload required files in Rails"
date: 2013-01-10 12:34
comments: true
categories: ruby

---

When a Rails project grows, I often notice the need to **refactor domain logic into a separate module, isolated from Rails framework**.

The isolated module will still stay in the main Rails app codebase, but can be easily packaged as a Ruby gem, tested separately, and used in other related applications.

## Requirements

During the refactoring, we want to:

1. **Use `require` to load the module like a normal gem**. If possible, no dependency on Rails autoload features such as`require_dependency` and `autoload_paths`.
   
2. **Edit the module without restarting server during development**. In other words, we need to find a way to **reload the module on every request**.

## Reload `require` files
After some research, I found a [**working solution**](http://timcardenas.com/automatically-reload-gems-in-rails-327-on-eve) by Timothy Cardenas:

```
ActionDispatch::Callbacks.to_prepare do
  if Object.const_defined?("Module1")
    Object.send(:remove_const, "Module1")
  end
  $".delete_if {|s| s.include?('module1') }
  require 'module1'
end
```

What it does is, before each request,

1. Unload the top-level module.
2. Un-require all required files from the module.
3. Re-require the top-level module.

## An extra step in Rails 3.2

In Rails 3.2, `ActionDispatch::Callbacks.to_prepare` has a slightly different behavior. It will run before a request **only if a watchable file is modified**. 

You need to specify your own [watchable files](http://api.rubyonrails.org/classes/Rails/Railtie/Configuration.html#method-i-watchable_dirs):

```
# watch all .rb files recursively under modules/module1/ dir
config.watchable_dirs['modules/module1'] = [:rb]
```

## Introduce `require_reloader` gem

Before bundling the solution into a gem, I did a search on RubyGems.org and found Colin Young's [gem_reloader](https://github.com/colinyoung/gem_reloader). It's based on Timy's solution as well. I forked it and started playing around. 

In the end, I made some major changes to include Rails 3.2 support, some fixes and new features.

So I decided to **release it as a new Ruby gem: [require_reloader](https://github.com/teohm/require_reloader)**.

```
# config/environments/development.rb
YourApp::Application.configure do
  ...
  RequireReloader.watch :module1,
    path: 'modules/module1'
end
```

Currently it supports Rails 3, including 3.1 and 3.2. 

If you are working on something similar, looking forward for your feedbacks and pull requests. It is not tested on Rails 2 yet, so pull requests are definitely welcomed!