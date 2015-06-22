---
layout: post
title: TDD your ruby gem a migration generator
modified: 2015-06-21
tags: [tutorial, ruby, gems, tdd]
comments: true
date: 2015-06-21T10:13:19-07:00
---

I maintain [a ruby gem][1] that was without a database migration generator for
far too long. This is a walkthrough on how I [TDD][2]'d a generator to create
its database migration generator.



## What's a migration generator?



A migration generator is the composition of ActiveRecord migrations within
Rails generators.  They are commonly found in Rails engines and plugins.



[Rails generators][3] are convenience scripts. Most frequently generators are
used to create or insert boilerplate code into an application.  

{% highlight bash %}

# An example Rails generator that will
# generate a new UsersController
#
$ rails generate controller User

{% endhighlight %}



[ActiveRecord migrations][4] are Ruby classes that define a version of the
application's database. Where each additional database migration only defines
how to incrementally modify the database from the previous version.

{% highlight bash %}

# Example database migrations to add a column to an existing model
#
$ rails generate migration add_email_to_users email:string

{% endhighlight %}



Now if we were to combine both of these concepts we arrive at migration
generators. Which are generators that are capable of creating database migrations.

{% highlight bash %}

# Devise will create a database migration for the Users table
#
$ rails generate devise User

# Creates a database migration to support my
# double-entry accounting gem
#
$ rails generate double_double:install

{% endhighlight %}



### TDD approach



For the purpose of this generator, I am most concerned that the database
migration template is copied into the expected location.

_We could further inspect the generated migration for its specific content,
but that's approaching overkill.  As other tests utilize the database, those
tests will certainly fail loudly if there is a problem with our migration._

To assist in our tests we will be using the [generator_spec][5] gem.  Let's
add it to our `.gemspec`.  Bundle install.

{% highlight ruby %}

# double_double.gemspec

Gem::Specification.new do |gem|
  # ...
  gem.add_development_dependency 'generator_spec'
end

{% endhighlight %}



### Our test



The skeleton of our Rspec spec:

{% highlight ruby %}

# spec/lib/generators/double_double/install/install_generator_spec.rb

require "generator_spec"

module DoubleDouble
  module Generators
    describe InstallGenerator, :type => :generator do

      root_dir = File.expand_path("../../../../../../tmp", __FILE__)
      destination root_dir

      before :all do
        prepare_destination
        run_generator
      end
    end
  end
end

{% endhighlight %}

Our most significant decision so far was deciding where to temporarily store
the generated files and directories.  The `tmp` directory is an obvious choice.

Let's continue and add our test case.

{% highlight ruby %}

# spec/lib/generators/double_double/install/install_generator_spec.rb

require "generator_spec"

module DoubleDouble
  module Generators
    describe InstallGenerator, :type => :generator do

      root_dir = File.expand_path("../../../../../../tmp", __FILE__)
      destination root_dir

      before :all do
        prepare_destination
        run_generator
      end

      it "creates the installation db migration" do
        migration_file = 
          Dir.glob("#{root_dir}/db/migrate/*create_double_double.rb")

        assert_file migration_file[0], 
          /class CreateDoubleDouble < ActiveRecord::Migration/
      end
    end
  end
end

{% endhighlight %}

**Now** we have a test to code against.  This test will assert:

- That the migration exists
- That some text in the migration is the classname that we expect



### Production code



_Fast-forward some TDD and a few cups of coffee..._


We wrote a proper database migration template.

{% highlight ruby %}

# lib/generators/double_double/install/templates/create_double_double.rb

class CreateDoubleDouble < ActiveRecord::Migration
  def change
    # ...
  end
end

{% endhighlight %}


And to finish the task we added the actual Rails generator.

{% highlight ruby %}

# lib/generators/double_double/install/install_generator.rb

require 'rails/generators'
require 'rails/generators/migration'

module DoubleDouble
  module Generators
    class InstallGenerator < ::Rails::Generators::Base
      include Rails::Generators::Migration
      source_root File.expand_path('../templates', __FILE__)
      desc "Add the migrations for DoubleDouble"

      def self.next_migration_number(path)
        next_migration_number = current_migration_number(dirname) + 1
        ActiveRecord::Migration.next_migration_number(next_migration_number)
      end

      def copy_migrations
        migration_template "create_double_double.rb",
          "db/migrate/create_double_double.rb"
      end
    end
  end
end

{% endhighlight %}

You may be asking: What's the class method `next_migration_number`?

It adds the date and time string to the migration filename. [I found that exact method implementation in Rails][6], itself.



[1]: http://github.com/crftr/double_double
[2]: https://en.wikipedia.org/wiki/Test-driven_development
[3]: http://guides.rubyonrails.org/generators.html
[4]: http://guides.rubyonrails.org/active_record_migrations.html
[5]: https://github.com/stevehodgkiss/generator_spec
[6]: https://github.com/rails/rails/blob/3e36db4406beea32772b1db1e9a16cc1e8aea14c/activerecord/lib/rails/generators/active_record/migration.rb