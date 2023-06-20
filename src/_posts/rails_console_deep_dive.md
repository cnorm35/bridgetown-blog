---
layout: post
title:  Rails Console Deep Dive
date:   2023-06-19 16:54:46 -0500
category: ruby
excerpt: "Tips and tricks to help you get the most from the Rails console."
author: cody
---
I received some great feedback on how newer Rails developers can start to
level up and get the most from the Rails console. I also heard from some
experienced developers they picked up some new tricks as well.  This will be a
more in-depth look at how you can get more out of the Rails console.

I'm not sure if it's true, but it definitely _feels_ like I spend a big chunk of
my dev time with my Rails console. I really love being able to check things in
the console prior to running the app and have a chance to spot potentials errors
before having to load the app. This is one of the ways I keep my feedback loop
tight.

Here are some handy tips and tricks to get the most from the Rails console.

### Helper methods

If you read my other post, _5 Tips for New Rails Developers_, some of these
may look familiar.

I love being able to use and test helper methods in the console.  Some of the
more common ones I find myself using are [ActionView helper methods](https://guides.rubyonrails.org/action_view_helpers.html){:target="_blank"} and I18n translations.
Quickly testing different options in the console is a much easier process without
having to refresh your browser and check the output.

Some examples from `ActionView::Helpers::NumberHelper`
[more info](https://api.rubyonrails.org/classes/ActionView/Helpers/NumberHelper.html){:target="_blank"}

```ruby
irb(main):001:0> helper.number_to_currency(123)
=> "$123.00"

irb(main):002:0> helper.number_to_currency('123', precision: 0)
=> "$123"

irb(main):004:0> helper.number_to_phone('5555555555')
=> "555-555-5555"

irb(main):005:0> helper.number_to_phone('5555555555', area_code: true)
=> "(555) 555-5555"

irb(main):006:0> helper.number_to_phone('5555555555', country_code: 1)
=> "+1-555-555-5555"


irb(main):007:0> helper.number_to_phone('5555555555a', raise: true)
/Users/cody/.rbenv/versions/3.2.2/lib/ruby/gems/3.2.0/gems/actionview-7.0.5/lib/action_view/helpers/number_helper.rb:453:in `parse_float': ActionView::Helpers::NumberHelper::InvalidNumberError (ActionView::Helpers::NumberHelper::InvalidNumberError)

```

If these look cool to you it's because you're absolutely right and you have
excellent taste.  There aren't a ton of problems that someone has never had to solve and Rails
usually has a solution or method for your need. Don't forget to check the
documentation  before implementing everything on your own.

While not specifically a helper method, checking I18n output, especially if
any options are being used, is another handy way to check things prior to
running your application.

```ruby
irb(main):001:0> I18n.t('users.agreements.show')
=> {:title>"%{agreement} Changed", :last_updated=>"Last updated %{date}", :description=>"Before you can proceed, you must read and accept the new %{agreement}.", :accept=>"I Accept", :decline=>"I Decline"}

irb(main):001:0> I18n.t('are_you_sure')
=> "Are you sure?"
```


### Checking for a Class or Module

After creating a new Class or Module, I will enter the name of my newly created
file and make sure it returns itself.  This is an easy way to check if your new
Class or Module has been loaded.

```ruby
irb(main):001:0> TwilioService
=> TwilioService

irb(main):002:0> FakeService
(irb):2:in `<main>': uninitialized constant FakeService (NameError) FakeService
```

### Viewing encrypted credentials

If you're using encrypted credentials and would like to see the values of your
credentials without needing to open your config file outside your console, we have some options for that in the Rails console.

These commands will return a full `Hash` of the credentials in your `yml` file.

There's nothing special about the object that's returned. It's just a regular
ruby Hash and you can access the values with the normal methods.

```ruby
irb(main):001:0> Rails.application.credentials
=> {...super secret...}

irb(main):002:0> Rails.application.credentials.dig(:twilio)[:auth_token]
=> '123123123'
```


### Setting the last result to a variable

This is one I hardly, if ever, use.  That's not because it
isn't useful. It's mainly because I have have a different (less fancy looking)
way of accomplishing it.

In the Rails console, there's a special method for returning the last value
returned with `_` 

```ruby
User.first
=> User stuff

user = _

=> <User ...>
```

While not as elegant, I usually hit the up key to get the last command, hit
`ctrl + a` to jump to the beginning of the line and save to a variable.

### Different methods of starting a Rails console

There are also some handy tips to know before even _starting_ your Rails console.

By default, running `rails console` starts up a Rails console within the context
of your `development` environment.

Let's say you have a bug or error on your `production` or `staging` environment.
If you don't feel like going full code-cowboy right off the bat, you can use the `-e` flag to specify a specific environment you would like to
start your console with.

You obviously wont have access to the DB data, but
being able to check environment variables and specific configurations for different
environments is a big help.

Sidenote: Using the Rails encrypted credentials makes this a much easier
process.

```bash
$ bin/rails console -e production
```

You're also not only limited to `test`, `development` or `production`.  Any
environment you've configured a file for within `config/environments/*` will be
available.

For example, if I have a staging environment configured for my app
and a file at `config/environments/staging.rb` or `config/environments/demo.rb` I could open a Rails console in
the context of the staging or demo environment with

```
$ bin/rails c -e staging
or
$ bin/rails c -e demo
```


### Starting your console in a sandbox

If you haven't heard the term 'sandbox' in terms of software development, it is
used to refer to a safe environment for your to experiment without impacting
real data.

Rails provides an option to start a console in a sandbox environment.  All
changes will be rolled back once you exit the console.

```
bin/rails c -e staging --sandbox
```

### Auto Complete Options
If you're on more recent versions of Rails, you may have noticed to autocomplete
feature in the console.  This seems to be a divisive feature.  Some people hate
it, I actually really appreciate that I can see all the available methods on a
given object.

At least...when I'm on my computer at home in a development environment.

If you need to access a rails console for your production environment,
it may feel sluggish and cause some memory issues.  _Especially_ if you're on a
server without a ton of memory.

There are options to add a line in your `.irbrc` to completely disable, but I
usually only want that in certain cases.  You may also not have access or
ability to add an `.irbrc` file on your server.

If you prefer to disable it all the time you can add this line to your `.irbrc`
file `IRB.conf[:USE_AUTOCOMPLETE] = false`

More [IRB options](https://docs.ruby-lang.org/en/master/IRB.html){:target="_blank"}


Rails provides a handy flag to pass to disable the autocomplete, _for that
console session_  This is a great way to fire up a rails console without all the
additional overhead of autocomplete while still having it available.

```
bin/rails c -- --noautocomplete
```

### Reloading

There are a couple of different options for reloading your Rails console.  If
you have a ton of variables set, or some other data you'd rather not blow away
by exiting and restarting a console, you can use:

```

irb(main):001:0> reload!
```

To update the code loaded into the console.  Say you added a new class you'd
like to access in your console.  You should be able to use `reload!` to update
the console and make the newly added class available.

If you need to reload an object, like a User record you fetched from the
database, you can call `.reload` on the object to re-fresh it's current state.
You'll see a new call to the database to fetch an updated version of your
object.

```
irb(main):001:0> user.reload
User Load (0.3ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1 LIMIT $2  [["id", 1], ["LIMIT", 1]]
```

### Route helpers
You can also check the paths for your route helpers that will return the path
for the given route by calling the route on the `app` object

```ruby
irb(main):001:0> app.new_billing_address_path
=> "/billing_address/new"

irb(main):002:0> app.root_path
=> "/"

```
### Source location

One of my favorite tricks to use in the Rails console is `source_location`.
[Source location](https://ruby-doc.org/3.2.2/Method.html#method-i-source_location){:target="_blank"} tells you the location where a method is defined.

Let's say we have a `User` model using Devise and requires the User to confirm their email
address.  If we'd like to find where the `send_confirmation_instructions` method
is defined we can use the code from the snippet below.

```ruby
irb(main):001:0> User.instance_method(:confirm).source_location
=> ["/Users/cody/.rbenv/versions/3.2.1/lib/ruby/gems/3.2.0/gems/devise-4.8.1/lib/devise/models/confirmable.rb", 79]
```

We can also use an instance of the User class to find where our method is
defined.

```ruby
irb(main):001:0> user.method(:send_confirmation_instructions).source_location
=> ["/Users/cody/.rbenv/versions/3.2.1/lib/ruby/gems/3.2.0/gems/devise-4.8.1/lib/devise/models/confirmable.rb", 115]
```


With this info, we can run `bundle open devise` (in a terminal window, not your
Rails console) and can look in
`lib/devise/models/confirmable.rb` on line 115 to see where the
`send_confirmation_instructions` method is defined.

That's a great option, but when I have to dig through source code, I usually use
GitHub since it's easier to share a link to a specific line in a file.


### Viewing database tables

Return an array of table names with

```ruby
irb(main):001:0> ActiveRecord::Base.connection.tables
```
If you need to do more than just see a list of the tables, you can use the handy
`rails dbconsole` to open a database console connected to the database
you have configured in you database.yml file.

If you've never entered into the database console, you can use `\q` to exit the
console session.

You'll see some errors if you haven't set up your database yet so make sure
that's been completed before trying it.  Or don't I'm not a cop (shrug)
If you've never used a DB console, at least on PG, use `/q` to exit.

I hope found some of these tips helpful. The Rails console is
absolutely one of my favorite tools in the rails ecosystem. Knowing the
capabilities and options available to you, helps you get the most from your tooling. It keeps your feedback loop small and fast and makes you look really
cool. Or, at least cool to me.
