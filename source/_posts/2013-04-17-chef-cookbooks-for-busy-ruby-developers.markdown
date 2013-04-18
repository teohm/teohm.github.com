---
layout: post
title: "Chef cookbooks for busy Ruby developers"
description: "In this post, I will show you a step-by-step guide on using the cookbooks together with knife-solo to provision a remote server in 4 steps."
date: 2013-04-17 23:06
comments: true
keywords: chef, chef-solo, ruby, rack, rackbox, databox, appbox, unicorn, passenger, rails.
categories:
- chef
- ruby

---

Have you ever **setup a Rails production environment from scratch, by
hand**? If you had, I share your pain every time when a new project
started.

The process is often repetitive. To me, it seems to be a waste to do it
manually every time. It also consumes time and attention. It would be
great if I could spend them on tasks that bring more values to clients.

To minimize such waste, I have written two Chef cookbooks to **automate
the process**:

* [**rackbox**](https://github.com/teohm/rackbox-cookbook)  - to provision **rack-based web server** (Nginx as front server, Unicorn and Passenger as upstream app servers, `rbenv` as ruby version manager).
* [**databox**](https://github.com/teohm/databox-cookbook) - to provision **database server** (supports MySQL and PostgreSQL).

Getting started
----------

In this post, I will show you a **step-by-step guide** on how to use the
cookbooks together with
[`knife-solo`](https://github.com/matschaffer/knife-solo) to provision a
remote server in 4 steps:

1. setup Chef Solo environment
2. modify config file
3. provision remote server
4. tweak Capistrano `deploy.rb`

A working example in also available at
[teohm/kitchen-example](https://github.com/teohm/kitchen-example).

1. Setup Chef Solo environment
-------------

* Install Chef Solo tools on local machine.
* Download Chef cookbooks to local machine.
* Install `chef-solo` on remote server.


### Install Chef Solo tools

Let's create a new directory,

```
mkdir chef-kitchen
cd chef-kitchen
```

and a `Gemfile`.

```
source "https://rubygems.org"

gem "knife-solo", ">= 0.3.0pre3"
gem "berkshelf"
```
   
I recommend `knife-solo >= 0.3` as it includes a few [major fixes and
improvements](https://github.com/matschaffer/knife-solo/blob/master/CHANGELOG.md#changes-and-new-features).
   
Now, install the ruby gems.

```
bundle install
```

Finally, setup a kitchen directory structure with `knife-solo`.

```
bundle exec knife solo init .
```

### Download Chef cookbooks

I use [Berkshelf](http://berkshelf.com/) to manage cookbooks. So we need a `Berksfile`,

```
site :opscode

cookbook "runit", ">= 1.1.2"  # HACK: force-use this version
cookbook "databox"
cookbook "rackbox"
```

(I added a hack here to force `berkshelf` to use `runit 1.1.2` required
by `rackbox`. Still looking for a better solution.)
   
We can now download cookbooks with `berks install`.

```
bundle exec berks install --path cookbooks/
```

### Install `chef-solo` on remote server

```
bundle exec knife solo prepare testbox
```

2. Customize config file
--------------

* Download config example
* Customize config file

### Download config example

```
curl https://raw.github.com/teohm/kitchen-example/master/nodes/host.json.example --output nodes/testbox.json
```

### Modify config file (JSON)

The config file starts with a `run_list`. You specify a list of cookbook
recipes here. Chef will run them in the same order in this list.

It is followed by cookbook attributes. You can modify these attributes.
A full reference of attributes are described in each cookbook's README
(see [appbox](https://github.com/teohm/appbox-cookbook#attributes),
[databox](https://github.com/teohm/databox-cookbook#attributes),
[rackbox](https://github.com/teohm/rackbox-cookbook#attributes)).

```
{
  "run_list": [
    "databox",
    "rackbox"
  ],
  "appbox": {
    "deploy_keys": ["ssh-rsa 5bnmu23890fghghjk"],
    "admin_keys": ["ssh-rsa 456789fghjkvbn567"]
  },
  "databox": {
    "db_root_password": "welcome!",
    "databases": {
      "mysql": [
        { "username": "app1",
          "password": "app1",
          "database_name": "app1_production" }
      ],
      "postgresql": [
        { "username": "app2",
          "password": "app2",
          "database_name": "app2_production" }
      ]
    }
  },
  "rackbox": {
    "ruby": {
      "versions": ["1.9.3-p385", "1.9.2-p320"],
      "global_version": "1.9.3-p385"
    },
    "apps": {
      "unicorn": [
        { "appname": "app1",
          "hostname": "app1.test.com" }
      ],
      "passenger": [
        { "appname": "app2",
          "hostname": "app2.test.com" }
      ]
    }
  }
}

```

3. Provision remote server
---------

```
bundle exec knife solo cook testbox
```

It uploads the kitchen directory and runs `chef-solo` on the remote
server. Chef-solo will then takeover and execute the run list to setup
everything.

### What do we get at this point?

Basically, it's done! 

We have a **full-stack, rack-based server** with:

* 3 user accounts (deploy, devops, apps)
* `rbenv` as ruby version manager
* `nginx` as front-server
* `unicorn`, `passenger-standalone` as upstream app servers, managed by `runit`
* `postgresql`, `mysql` installed and databases created
*  all apps will be stored in `/home/apps/`

4. Tweak Capistrano deploy.rb
-------------

Now, it's ready to deploy a Rack-based app to the remote server!

I have two example Rails apps available on Github:

* [teohm/sample-app1](https://github.com/teohm/sample-app1) runs on unicorn,
* [teohm/sample-app2](https://github.com/teohm/sample-app2) runs on passenger-standalone. 

There are a few **minor tweaks required in Capistrano `deploy.rb`**, as listed below. 

I will explain the tweaks in details next time. Meanwhile, check out the complete working examples at: [app1/config/deploy.rb](https://github.com/teohm/sample-app1/blob/master/config/deploy.rb) and [app2/config/deploy.rb](https://github.com/teohm/sample-app2/blob/master/config/deploy.rb)


### Login as `deploy` user
```
set :user, "deploy"
```

### Deploy to `/home/apps`
```
set :deploy_to, "/home/apps/#{application}"
```

### Load `rbenv` in Capistrano
```
default_run_options[:shell] = '/bin/bash --login'
```

### Run bundler with `--binstubs`
```
require 'bundler/capistrano'
set :bundle_flags, "--deployment --binstubs"
set :bundle_without, [:test, :development, :deploy]
```

### Restart app with `runit sv`
```
namespace :deploy do
  task :start do
    run "sudo sv up app1"
  end
  task :stop do
    run "sudo sv down app1"
  end
  task :restart, :roles => :app, :except => { :no_release => true } do
    run "sudo sv restart app1"
  end
end

```

Feedback
-------
If you are interested on using the cookbooks, or have an idea/feedback/question about this topic, feel free to drop me ([@teohm](http://twitter.com/teohm)) a message. Pull requests and issue reports are definitely welcomed!

