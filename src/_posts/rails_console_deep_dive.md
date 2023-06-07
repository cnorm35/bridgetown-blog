---
layout: post
title:  Rails Console Deep Dive
date:   2023-06-02 16:54:46 -0500
category: ruby
excerpt: "Tips and tricks to help you get the most from your Rails console"
author: cody
---
In a recent post, I received a lot of great feedback on some of the items,
especially the tips about how to get the most out of your Rails console.  It was
great seeing new and experienced developers pick up a couple of new tips to add
to their arsenal.

I'm not sure if it's true, but it defintely _feel_ like I spend a big chunk of
my dev time with my Rails console.  I really love being able to check things in
the console prior to running the app and seeing an error page.

This is one of my tips for keeping your feedback loop tight.  I'd rather see an
error or spot an issue within my console instead of the Rails error page.  Don't
get me wrong, the error page is great and having a detailed stacktrace and live
console already loaded with the state of your application is a huge benefit.
But, in keeping with the unix philospphy of focused tooling, your error page is
not where you want to be spending your time.

I'm going to go through some of my favorite things to do in the Rails console.  


If you read my other post with 5 ti[s for new Rails developers, some of these
may look familiar.

### Helper methods

This is something I use pretty frequently.  Some particular things I find myself
checking a lot are view helper methods and I18n translations.  Those are a
couple of things I find myself forgetting a bit of the syntax and options.
Running these methods in your console let you test output and play around with
some of the different options you may be able to pass.

```ruby
helper.number_to_currency(123)
=> "$123.00"

irb(main):002:0> helper.number_to_currency('123', precision: 0)
=> "$123"

irb(main):004:0> helper.number_to_phone('5555555555')
=> "555-555-5555"

irb(main):005:0> helper.number_to_phone('5555555555', area_code: true)
=> "(555) 555-5555"

irb(main):006:0> helper.number_to_phone('5555555555', country_code: 1)
=> "+1-555-555-5555"


irb(main):010:0> helper.number_to_phone('5555555555a', raise: true)
/Users/cody/.rbenv/versions/3.2.2/lib/ruby/gems/3.2.0/gems/actionview-7.0.5/lib/action_view/helpers/number_helper.rb:453:in `parse_float': ActionView::Helpers::NumberHelper::InvalidNumberError (ActionView::Helpers::NumberHelper::InvalidNumberError)

```
If these look cool to you it's because you're absolutely right.  There aren't a
ton of unique problems that people have never had to solve before and Rails
usually has a soultion or method to help so don't forget to check around before
implementing everything on your own.

https://api.rubyonrails.org/classes/ActionView/Helpers/NumberHelper.html

While not specifically a helper method, checking I18n output, especially if
there are any options passed is another handy way to check things prior to
running.

Helper Methods and how to call them
I18n stuff


Another thing I like to check is making sure new Modules and Classes I've added
are loaded correctly


```ruby
irb(main):001:0> TwilioService
=> TwilioService
irb(main):002:0> FakeService
(irb):2:in `<main>': uninitialized constant FakeService (NameError)

FakeService
```

If you're using encrypted credentials and would like to see the values of your
creds without needing to open your config file ouside your console, you can use 


```
Rails.application.credentials
```

Which will return a full Hash of the credentials in your yml file.

There's nothing special about the object that's returned, it's just a regular
ruby Hash and you can access the values with the normal methods.



```
Rails.application.credentials.dig(:omniauth, provider.to_sym)
```


### Setting the last result to a variable

This is one I hardly, if ever use if I'm being honest.  That's not because it
isn't useful, mostly that's because I have have a different (less fancy looking)
way of accomplishing it.

In the Rails console, there's a special method for returning the last value
returned with the `_`method

```ruby
User.first
=> User stuff

user = _

=> user stuff returned.
```

While not as elegant, I usually hit the up key to get the last command, hit
crt+i (or whatever it is) to jump to the beginning of the line.

Also, somewhere along the way, I found that most of the variables I set in the
console are going to be compared to another variable.  I pretty much always use
`something` for my subject and `something_else` for the comparison.  Nothing
fancy, but have a rule around that is one less thing to think about.  Keeping
brain friction to a minimum saves that brain power for your more important tasks
instead of trying to decide what you're going to name a variable



### Different methods of starting a Rails console

There are also some handy options for _starting_ your Rails console.

By default, running `rails console` starts up a Rails console within the context
of your development environment.

Let's say you don't feel like going full code-cowboy and trying to fix an issue
on your production or staging environment.

you can use the `-e` flag to specify a specifc environment you would like to
start your console with.  You obviously wont have access to the DB data, but
being able to check environment vars and specific configs for different
environments is a big help.

```bash
$ bin/rails console -e production
```

You're also not only limited to `test`, `development` or `production`.  Any
environment you've configured a file for within `config/environments/` will be
available.  For example, if I have a staging environment configured for my app
and a file at `config/environments/staging.rb` I could open a Rails console in
the context of the staging environment with

```
bin/rails c -e staging
```


### Starting your console in a sandbox


If you haven't heard the term 'sandbox' in terms of software development, it is
used to refer to a safe environment for your to experiment without impacting
real data.

Rails provides this option for our console and is able to be appended to your
command to incliude this with the above command

```
bin/rails c -e production --sandbox
```

Rails will automatically rollback any changes to the data you make during the
session so can be a good method for exploring things.

If you're on more recent versions of Rails, you may have noticed to autocomplete
feature in the console.  This seems to be a devisive feature.  Some people hate
it, I actually really appreicate that I can see all the available methods on a
given object.

That is, when I'm on my computer at home in a development environment.
However, if you need to access a rails console for your production environment,
it may feel sluggish and cause some memory issues.  Especially if you're on a
server without a ton of memory.

There are options to add a line in your `irbrc` to completly disable, but I
usually only want that in certain cases.  You may also not have access or
ability to add an .irbrc file on your server.

Rails provides a handy flag to pass to disable the autocomplete, _for that
console session_  This is a great way to fire up a rails console without all the
additional overhead of autocomplete while still having it available.

```
bin/rails c -- --noautocomplete
```


### Reloading

There are a couple of different options for reloading your rails console.  If
you have a ton of variables set, or some other data you'd rather not blow away
by exiting and restarting a console, you can use 

```
reload!
```

To upaate the code loaded into the console.  Say you added a new class you'd
like to access in your console.  You should be able to use `reload!` to update
the console and make the newly added class available.

If you need to reload an object, like a User record you fetched from the
database, you can call `.reload` on the object to re-fresh it's current state.


https://docs.ruby-lang.org/en/master/IRB.html
(

IRB config options
IRB.conf[:USE_AUTOCOMPLETE] = false

Working version
`bin/rails c -- --noautocomplete`
<!-- Flags and options for starting the console (environments, sandbox, --no -->
<!-- autocomplete on production -->
<!-- clear console, reload object, reload! -->

<!-- value of last expression with `_` but I seriously just use something and -->
<!-- something_else as my var names.  Maybe say something about that Jason Sweet post -->
<!-- about making rules for not thinking about things -->

route stuff
<!-- encrypted creds -->
source_location stuff and source code for method
listing and searching for methods
talk about the `app` object


Touch on Rails runners and DBConsole

