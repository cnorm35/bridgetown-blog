---
layout: post
title:  5 tips for new Rails developers
date:   2023-06-02 16:54:46 -0500
category: ruby
excerpt: "5 tips for new Rails developers to be more productive and confident."
author: cody
---

I’ve been fortunate enough to have worked alongside a top-notch group of newer developers over the past couple of months.  It’s been the first time in a few years that I’ve regularly worked with new developers and has been a great chance to re-examine and question how I do a lot of things.

We conduct morning standups where everyone has a chance to discuss their issues they’re working on and any blockers they may have.  We run through a normal standup and then take the remaining time we have (usually from my free Zoom account, please don’t hate) we’ll try to review a recent issue or fix in greater detail or see if the group has any questions and would like to go into more detail.

These talks range from How [Concerns](https://api.rubyonrails.org/classes/ActiveSupport/Concern.html){:target="_blank"} work and how to create and use them.

Where is the best place for this test and what’s needed to set it up?

And even the occasional live coding session where I typically drive and talk about what I’m doing.  Rebasing a branch and cleaning up some commits is a typical example of that one.

It seems like every day I have a “Oh by the way…” that leads into some examples or tips I’ve learned, usually painfully, over the years.
This is by no means exhaustive, but here are a few tips I’ve found myself talking about a lot recently.

### 1. You should probably be using your Rails console more.
(add something about IRBrc expanding models here)
This one might be a little opinionated. I’m a hands-on learner and I'm _always_ poking around in the Rails console.  Getting a running Rails console and pulling a model is usually the first thing I do after getting a new app up I’m onboarding to and running.

You can also customize your Rails console by updating your `~/.irbrc` file.  Some of the things you can customize are automatically expanding objects or disabling the autocomplete available in the newer versions of Ruby.

Here are some examples of common things I do within the Rails console.

Testing the output of helper methods by using `helper`

```ruby
helper.nubmer_to_currency(123)
=> "$123.00"
```

Another pretty common scenario is checking I18n output making sure I'm using the
correct values and seeing the correct output

```ruby
irb(main):001:0> I18n.t('users.agreements.show')
=> {:title=>"%{agreement} Changed", :last_updated=>"Last updated %{date}", :description=>"Before you can proceed, you must read and accept the new %{agreement}.", :accept=>"I Accept", :decline=>"I Decline"}
```

A quick smoke test to make sure a new class or module has been added.  An example of this would be creating a concern called MySpecialConcern and calling that in the console.  If it’s defined correctly, it will return itself. Let's say I just added a new plain ruby class called `TwilioService` and would like to confirm it's being loaded correctly.  The main reasons it would not load correctly are misspellings either with the class or the file name or the location of the file not being autoloaded into your application.  It's just another quick test to catch errors quickly.

```ruby
irb(main):001:0> TwilioService
=> TwilioService
irb(main):002:0> FakeService
(irb):2:in `<main>': uninitialized constant FakeService (NameError)

FakeService
```

It's also pretty common for me to test out queries, test changes, confirm
validations, and several other checks I make before heading to my browser.

One of my favorite tricks to use in the Rails console is `source_location`.
[Source location](https://ruby-doc.org/3.2.2/Method.html#method-i-source_location){:target="_blank"} tells you the location where a method is defined.

Let's say we have a `User` model using Devise and requires the User to confirm their email
address.  If we'd like to find where the `send_confirmation_instructions` method
is defined we can use the code from the snippet below.

```ruby
irb(main):001:0> user.method(:send_confirmation_instructions).source_location
=> ["/Users/cody/.rbenv/versions/3.2.1/lib/ruby/gems/3.2.0/gems/devise-4.8.1/lib/devise/models/confirmable.rb", 115]
```

With this info, we can run `bundle open devise` and can look in
`lib/devise/models/confirmable.rb` on line 115 to see where the
`send_confirmation_instructions` method is defined.

That's a great option, but when I have to dig through source code, I usually use
GitHub since it's easier to share a link to a specific line in a file (more on
that below)

### 2. Practice using more debugging tools.
If you run your Rails application with bin/dev using something like Foreman,
this can be a little tricky for a newer developer.

In this scenario, you’ll place your debugger (pry, debugger, byebug, or whatever) and your server process will pause and display a debugger prompt that you’re unable to connect to.  In other words, you'll see the debugger prompt, but you're not able to interact with it.

I’m a big fan of [overmind](https://github.com/DarthSim/overmind) which will let me connect to a specific process in my Procfile.dev.  What does all that mean?  Let's say I start my Rails server in a process I’m calling ‘web’. I have other services that handle things like rebuilding js and css, and sometimes extras like stripe cli, docker, or ngrok.  Here's an example of what's in my `Procfile.dev`

```
web: bin/rails server -p $PORT
css: yarn build:css --watch
js: yarn build --reload
worker: bundle exec sidekiq
stripe: stripe listen --forward-to localhost:5000/webhooks/stripe
ngrok: ngrok http --log=stdout 5000
```

I can connect to any one of these individual processes with:

`$ overmind connect PROCESS_NAME`

If I placed a debugger in one of my controllers, I could connect to my web
process with `overmind connect web`

This connects me to my web process with overmind.  This is along the same lines as if I just started my Rails server with `rails s`.

If I wanted to do something similar to debug a job running in Sidekiq, I could
place a debugger in my job and connect to the `worker` process with `overmind
connect worker`

Overmind requires tmux, which don't get me wrong, is one of my favorite tools. But, it’s still an additional dependency and one more thing to install and manage as a newer dev.

I think the easiest way to get to a live debugger is something like commenting out the web process in your Procfile (so you can still use `bin/dev` for assets and other items if that's already set up), open a new window for your terminal and start a Rails server.  In this window, your debugger will be available.

Now that you have connected to your debugger, what's next?

Usually, I check the current state of variables and objects. Sometimes I’ll build new ones, call `.valid?`, and check any errors before trying to save.  That lets me try to set some values and see if I know what’s required to get our object to a valid state.

My debugger of choice is [ruby debug](https://github.com/ruby/debug) and my favorite unsung feature of ruby debug is it has an alias that allows you to use `debugger` which is the same keyword for the JS debugger.

There are a ton of options you can use.  I don’t use much more past ‘c’ for continue meaning continue execution, and ‘n’ for next when goes to the next line.

The biggest benefit for me is using it as a quick way to re-create the exact conditions of an error making for a much shorter feedback loop.  I'll have some more to say on keeping a short feedback loop.

### 3. Communication.
<!-- This is a pretty broad one. This can be anything from writing a good commit message, a good pull request description, comment for clarification on an issue, how to clarify feedback and approach and even how to talk about what you learned in more detail to share with others -->


This is a pretty broad one.  Ruby was designed with programmer happiness and
readability in mind.  It can be really easy to read through some ruby code and
have an idea of what's happening.  Readability and clarity are things you should
focus on outside of your code editor as well.  It can be intimidating to ask for
clarification or fill out a bug report on a new project.

To feel more confident in your communication, do a little bit of digging around
before sending your message.  What exactly does that mean?

Let's say you're not sure who or how to ask for clarification on a ticket you've
been assigned.  Look through some of the recently closed tickets to get an idea
of how that particular team is communicating and any important information you
need to include.  This can help you get together things like pull request links,
screenshots, or steps to re-produce beforehand, giving whoever you're reaching out
to the best information.

You can do the same thing with async communication tools like Slack and
Discord.  Skim through the history and see if your question has already been
answered or the best way to ask your question.

Before opening my first pull request on a team.  I'll take some time and
review some recently merged pull requests to make sure I have everything I need,
see if there's anything that I may need to include that may not be included in a
pull request template if there's even one at all.

Solid communication is something that makes it easier for others to give you help.  It's not
uncommon for me to work on a few projects within a single day.  Giving my poor,
context-switched brain detailed information on what you've tried, what you
expect to happen, and what's actually happening can give me a much better
context as to what's happening.

Good communication between technical and less technical stakeholders can be a
huge benefit.


### 4. Code Spelunking.
I don't think this is a real term, this is just something I refer to as
embarking into source code to find some more information.  Most of the time, this is in
the source code of a gem.

Being able to search through source code for gems you're using to see how or where things are defined, seeing how they’re implemented, and seeing examples of how it's being tested are all some examples of common things I'll do.

Searching the git blame for history on changes is another good one.  Viewing the
history and seeing each commit where each line was introduced can give some
great information on a file.  If you see 1 or 2 single commits, a lot of times,
those are bug fixes.  Clicking on that commit in Github takes you to all the
changes for that commit and a quick search of that commit will point you to the
pull request where that change was merged in.

Lots of single commits could indicate a lot of churn in that file, meaning there
are lots of smaller changes which could mean multiple bugs and issues. This is
some great infomration on a potentially brittle file that you may want to explore
further.  All it took was a few clicks.

Searching by a specific term or feature and filtering by closed pull requests to see what’s been merged is another way to get some more information on a task if you're not sure where to start.


### 5. Keep your feedback loop tight.
Failing fast isn't just something that's reserved for startups.  Failing fast in
development terms, at least for me, is trying or testing things after _each
change_.  Or, whenever it makes sense.  Doing this helps me see issues and
errors faster and more importantly, with fewer changes introduced since the last
working state.  Having a list of 1-2 things that could be wrong can help you
find your issue so much faster than if you have 10-15.

I can’t tell you how many times I’ve tried to implement a whole spec complete with `context`, `describe`, and `before` blocks of setup only to leave out an end somewhere.

A good habit I always recommend is adding your blocks one at a time and running your spec to make sure everything is still working.

A few extra seconds spent checking and finding errors fast after minimal changes beats commenting out and running things to try to get it all working.

**Run often and fail fast.**

Like I said, this is by no means exhaustive or definitive, just some things I’ve found myself talking about on a few occasions recently.  If there are some tips you’ve found useful for becoming a more comfortable and confident Rails developer, please let me know, I’d love to hear them.

I hope the tips are helpful for you and help you feel more comfortable with
Rails.  Let me know which tips you think are the most helpful, I'll be expanding
on each one of these in more detail in some future posts.

Please tell me about some of your favorite tips I left out!

