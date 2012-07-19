---
layout: post
title: Using JQuery Validation in Rails Remote Form
description: >
  This post describes how to integrate JQuery Validation's 
  submitHandler(form) with Rails 3 remote form.
comments: true
keywords: rails, jquery, input validation, remote form, ruby, jquery UJS.
categories: 
- rails
- rails-view
- rails-frontend
---

In a recent project, I was trying to use [JQuery Validation](http://bassistance.de/jquery-plugins/jquery-plugin-validation/) in an [earlier version](https://github.com/rails/jquery-ujs/blob/bb620a29199214b5c7a3a015372fda28b69c69a1/src/rails.js) of Rails 3 remote form ([jquery-ujs](https://github.com/rails/jquery-ujs)). They didn't work out well in IE.

After experimenting with the [latest jquery-ujs](https://github.com/rails/jquery-ujs/blob/dad6982dc592686677e6845e681233c40d2ead27/src/rails.js) and making an embarrassing mistake, it turns out that the issue is resolved in the latest version.

(*Mistake*: You may notice I removed a previous post about this topic, where I mistakenly concluded the latest jquery-ujs is not working with JQuery Validation. Thanks to JangoSteve for [pointing it out](https://github.com/rails/jquery-ujs/issues/165#comment_1228899). The post was misleading, so I believe it's best to remove it to avoid confusion. :-)


Get the latest `jquery-ujs`
---
There are 2 reasons to use the latest jquery-ujs:

1. it has a patch that fixes the issue (see [issue #118](https://github.com/rails/jquery-ujs/issues/118)).
2. it exposes an internal function that we may need -- `$.rails.handleRemote()` (see [more details](http://www.alfajango.com/blog/rails-jquery-ujs-now-interactive/))


Working example
--
The example is tested with:

- JQuery 1.6.1
- JQuery Vaidation 1.8.1
- Latest Rails UJS (commit: [dad6982dc592686677e6](https://github.com/rails/jquery-ujs/blob/dad6982dc592686677e6845e681233c40d2ead27/src/rails.js))

### When using `submitHandler` in JQuery Validation

If you are using JQuery Validation's `submitHandler(form)` function, you need to 
call `$.rails.handleRemote( $(form) )` function manually, so that it submits the form via XHR.

{% highlight javascript %}
$('#submit_form').validate({

  submitHandler: function(form) {
    // .. do something before submit ..
    $.rails.handleRemote( $(form) );  // submit via xhr
    //form.submit();                  // don't use, it submits the form directly
  }

});
{% endhighlight %}
