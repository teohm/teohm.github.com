---
layout: post
title: Using RABL in Rails JSON Web API
description: >
  This post explains how to use RABL effectively to expose
  JSON Web API for a simple event management app. Sample code included.
comments: true
image: /images/tags/ruby-rails.png
keywords: rails, rabl, ruby, json, view renderer, web api, example. 
categories: 
- rails
- rails-view
---

Let's use an event management app as the example. 

The app has a simple feature: a user can add some events, then invite other users to attend the event. Its data are represented in 3 models: User, Event, and Event Guest.
![An ER digram that shows the domain models.](/images/2011-05-31-using-rabl-in-rails-json-web-api/models.png)

Let say, we are going to add a read-only JSON web API for clients to browse data records from the app.

## Problems

### Model is not view
When working on a non-trivial web API, you will soon [realize](http://engineering.gomiso.com/2011/05/16/if-youre-using-to_json-youre-doing-it-wrong/) that, model often cannot be serialized directly in web API. 

Within the same app, one API may need to render a summary view of the model, while another needs a detail view of the same model. You want to serialize a view or view object, not a model.

[**RABL (Ruby API Builder Language) gem**](https://github.com/nesquena/rabl) is designed for this purpose.


### Define once, reuse everywhere
Let say, we need to render these user attributes: `id`, `username`, `email`, `display_name`, except `password`.

In RABL, we can define the attribute whitelist in a RABL template.
{% highlight ruby %}
# tryrabl/app/views/users/base.rabl
attributes :id, :username, :email, :display_name
{% endhighlight %}

To show individual user, we can now reuse the template through RABL `extends`.
{% highlight ruby %}
# tryrabl/app/views/users/show.rabl
extends "users/base"
object @user

## JSON output:
# {
#     "user": {
#         "id": 8,
#         "username": "blaise",
#         "email": "matteo@wilkinsonhuel.name",
#         "display_name": "Ms. Noe Lowe"
#     }
# }
{% endhighlight %}

Here's another example to show a list of users.
{% highlight ruby %}
# tryrabl/app/views/users/index.rabl
extends "users/base"
collection @users

## JSON output:
# [{
#     "user": {
#         "id": 1,
#         "username": "alanna",
#         "email": "rubie@hayes.name",
#         "display_name": "Mrs. Gaylord Hoeger"
#     }
# }, {
#     "user": {
#         "id": 2,
#         "username": "jarrell.robel",
#         "email": "jarod@eichmann.com",
#         "display_name": "Oran Lebsack"
#     }
# }]
{% endhighlight %}

The template can be reused in nested child as well, through RABL `child`. 
{% highlight ruby %}
attributes :id, :title, :description, :start, :end, :location
child :creator => :creator do
  extends 'users/base'
end

## JSON output:
# {
#     "event": {
#         "id": 7,
#         "title": "Et earum sed fuga.",
#         "description": "Quis sed ..e ad.",
#         "start": "2011-05-31T08:31:45Z",
#         "end": "2011-06-01T08:31:45Z",
#         "location": "Saul Tunnel",
#         "creator": {
#             "id": 1,
#             "username": "alanna",
#             "email": "rubie@hayes.name",
#             "display_name": "Mrs. Gaylord Hoeger"
#         }
#     }
# }
{% endhighlight %}


### Join table rendered as subclass 

I notice a recurring pattern in two recent projects. For instance, in this example, from client's point of view, Event Guest is basically a User with an additional attribute: RSVP status.

When query database, usually we need to query the join table: `event_guests`.
{% highlight ruby %}
class GuestsController < ApplicationController
  def index
    @guests = EventGuest.where(:event_id => params[:event_id])
  end
end
{% endhighlight %}

But when rendering, the result set needs to be rendered as a list of Users. RABL allows you to do that easily, using its `glue` feature (a weird name though :).
{% highlight ruby %}
# tryrabl/app/views/guests/index.rabl
collection @event_guests

# include the additional attribute
attributes :rsvp

# add child attributes to parent model
glue :user do
  extends "users/base"
end

## JSON output:
# [{
#     "event_guest": {
#         "rsvp": "PENDING",
#         "id": 3,
#         "username": "myrna_harvey",
#         "email": "shad.armstrong@littelpouros.name",
#         "display_name": "Savion Balistreri"
#     }
# }, {
#     "event_guest": {
#         "rsvp": "PENDING",
#         "id": 4,
#         "username": "adelle.nader",
#         "email": "brendon.howe@cormiergrady.info",
#         "display_name": "Edgardo Dickens"
#     }
# }]
{% endhighlight %}


I think I will use RABL for the next Rails web API project.

The complete Rails example code is available at [github.com/teohm/tryrabl](https://github.com/teohm/tryrabl).
