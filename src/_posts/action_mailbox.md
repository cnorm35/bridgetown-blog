---
layout: post
title:  Action Mailbox
date:   2024-01-22 16:54:46 -0500
category: ruby
excerpt: "Getting started with Action Mailbox"
author: cody
---

Here is something about the post

Rough Topic List:

Routes and Routing

Creating a Catchall

Sending a basic email

Sending an email by source

Sending with Attachments

Parsing Attachments

Maill::Message and ruby mail

Gotchas with the email processing with ActiveJob

Can you use ActionMailbox outside of Rails? (if so talk about how to do it)


I've said it before and I'll say it again...

I love building email and sms features into my apps.

These are two channels that are comfortable and familar to your users with low
friction. 

This article is going to focus on the email portion of this. Specifically, using
ActionMailbox to allow incoming emails to your Rails app.

This post is going to walk through getting started with ActionMailbox, things to
consider on production and my experiences.

### What is Action Mailbox?

Let's face it, if you're like me, and you see some familar looking commands to
get started at the top of a guide, I can't resist running it just to see what it
does. 

If you're going through the Action Mailbox basics Rails guides, you could skip
over some important peices of information.

The Rails guides do a pretty good job of waht Action Mailbox is, so let's take a
look at that first
[What is Action Mailbox](https://guides.rubyonrails.org/action_mailbox_basics.html#what-is-action-mailbox-questionmark)


<blockquote>
  <cite>
    Action Mailbox routes incoming emails to controller-like mailboxes for processing in Rails. It ships with ingresses for Mailgun, Mandrill, Postmark, and SendGrid. You can also handle inbound mails directly via the built-in Exim, Postfix, and Qmail ingresses.

The inbound emails are turned into InboundEmail records using Active Record and feature lifecycle tracking, storage of the original email on cloud storage via Active Storage, and responsible data handling with on-by-default incineration.

These inbound emails are routed asynchronously using Active Job to one or several dedicated mailboxes, which are capable of interacting directly with the rest of your domain model.
  </cite>
</blockquote>

Breaking things down, ActionMailbox gives us an `InboundEmail` Active Record
object.

Two really important items to note are:

` storage of the original email on cloud storage via Active Storage...` and `These
inbound emails are routed asynchronously using Active Job...`

It's really easy to gloss over those requirements and can cause some issues when
deploying to production. I'll go into that in more detail when going over some
of my experiences deploying to production.

### Getting Started

To install ActionMailbox run the following commands

```
bin/rails action_mailbox:install
bin/rails db:migrate
```
That creates a ApplicationMailbox and a new migration for the InboundEmails.

```
 bin/rails action_mailbox:install
Copying application_mailbox.rb to app/mailboxes
      create  app/mailboxes/application_mailbox.rb
       rails  railties:install:migrations FROM=active_storage,action_mailbox
Copied migration 20240123213529_create_action_mailbox_tables.action_mailbox.rb from action_mailbox
```

```
# This migration comes from action_mailbox (originally 20180917164000)
class CreateActionMailboxTables < ActiveRecord::Migration[6.0]
  def change
    create_table :action_mailbox_inbound_emails do |t|
      t.integer :status, default: 0, null: false
      t.string  :message_id, null: false
      t.string  :message_checksum, null: false

      t.timestamps

      t.index [ :message_id, :message_checksum ], name: "index_action_mailbox_inbound_emails_uniqueness", unique: true
    end
  end
end
```

Running the migrate task and restarting your app and just like that you're able
to accept inbound emails to your app.  Sort of.

When we ran the initial install task, there was an
`app/mailboxes/application_mailbox.rb` file created.

```
class ApplicationMailbox < ActionMailbox::Base
  # routing /something/i => :somewhere
end
```

The `ApplicationMailbox` is where all of your inbound emails will be sent.  This
file decides which mailbox to route the email to.  Like any good Rails tool,
there's a task to generate a new mailbox.

```
bin/rails generage mailbox replies
```

Will create a new mailbox that inherits from ApplicationMailbox

```
class RepliesMailbox < ApplicationMailbox
  def process
  end
end
```

The `process` method is where the magic happens but first, we need to direct our
InboundEmail to the RepliesMailbox

There are a couple of options for directing the InboundEmail to the correct
mailbox.  What you'll be using in production (most likely) will be matching
emails that include `@replies` in the correct portion of the email address.

For development purposes, especially when you have a single Mailbox, I've found
it helpful to use `:all` to send _any_ inbound email to the mailbox you pass it.

`routing all: :replies`

This lets you start focuing on the business logic of the mailbox right away and
update the routing as your needs evolve.

Now, everything is setup to forward _any_ inbound email to the `RepliesMailbox`

You may be wondering...

How do I send an email to my local Rails App?

It's time to have your ticket punched by the Rails Condutor


### Sending Emails with Rails Conductor





```
# app/mailboxes/application_mailbox.rb
routing all: :replies
# routing(/@replies\./i => :replies)
```
