---
layout: post
title: TDD your ruby gem a migration generator
modified: 2015-06-19
tags: [tutorial, ruby, gems, tdd]
comments: true
date: 2015-06-19T16:13:19-07:00
---

I maintain [a ruby gem][1] that was without a database migration generator for
far too long. This is a walkthrough on how I [TDD][2]'d a generator to create
its database migration generator.



## What's a migration generator?



A migration generator is a composition of Rails generators and
ActiveRecord migrations.  They are commonly found in Rails engines and plugins.



[Rails generators][3] are convenience scripts. Most frequently generators are
used to insert boilerplate code into an application.  

{% highlight bash %}

# An example Rails generator that will
# generate a new UsersController

$ rails generate controller User

{% endhighlight %}



[ActiveRecord migrations][4] are Ruby classes that define a version of the
application's database. Where each additional database migration only defines
how to incrementally modify the database from the previous version.

{% highlight bash %}

# Example database migrations

# Bootstrap and generate the configuration to integrate Devise
$ rails generate devise:install

# Creates a database migration to support
# my double-entry accounting gem
$ rails generate double_double:install
{% endhighlight %}





{% highlight ruby %}
# Example can be run directly in your JavaScript console
def new_for_sure
  newbie = 1
  newbie += 2
end
{% endhighlight %}




Cum sociis natoque penatibus et magnis dis `code element` montes, nascetur ridiculus mus.

{% highlight js %}
// Example can be run directly in your JavaScript console

// Create a function that takes two arguments and returns the sum of those arguments
var adder = new Function("a", "b", "return a + b");

// Call the function
adder(2, 6);
// > 8
{% endhighlight %}

Aenean lacinia bibendum nulla sed consectetur. Etiam porta sem malesuada magna mollis euismod. Fusce dapibus, tellus ac cursus commodo, tortor mauris condimentum nibh, ut fermentum massa.

[1]: http://github.com/crftr/double_double
[2]: https://en.wikipedia.org/wiki/Test-driven_development
[3]: http://guides.rubyonrails.org/generators.html
[4]: http://guides.rubyonrails.org/active_record_migrations.html