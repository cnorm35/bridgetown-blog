---
layout: post
title:  Inbox Hero
date:   2024-01-22 16:54:46 -0500
category: ruby
excerpt: "Getting started with Action Mailbox"
author: cody
---

<!-- Here is something about the post -->

<!-- Rough Topic List: -->

<!-- Routes and Routing -->

<!-- Creating a Catchall -->

<!-- Sending a basic email -->

<!-- Sending an email by source -->

<!-- Sending with Attachments -->

<!-- Parsing Attachments -->

<!-- Maill::Message and ruby mail -->

<!-- Gotchas with the email processing with ActiveJob -->

<!-- Can you use ActionMailbox outside of Rails? (if so talk about how to do it) -->

<!-- Talk about `  before_processing :ensure_list` or something like that for before -->
<!-- hooks -->

<!-- Bounce With -->

<!-- before_processing -->


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

```ruby
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
InboundEmail to the RepliesMailbox.

There are a couple of options for directing the InboundEmail to the correct
mailbox.  What you'll be using in production (most likely) will be matching
emails that include `@replies` in the correct portion of the email address.

For development purposes, especially when you have a single Mailbox, I've found
it helpful to use `:all` to send _any_ inbound email to the mailbox you pass it.

`routing all: :replies`

```
# app/mailboxes/application_mailbox.rb
routing all: :replies
# routing(/@replies\./i => :replies)
```

This lets you start focuing on the business logic of the mailbox right away and
update the routing as your needs evolve.

Now, everything is setup to forward _any_ inbound email to the `RepliesMailbox`

You may be wondering...

How do I send an email to my local Rails App?

It's time to have your ticket punched by the Rails Conductor


### Sending Emails with Rails Conductor

http://localhost:3000/rails/conductor/action_mailbox/inbound_emails

[Screenshot]


One of my favorite hidden gems (yeah I said it) of Rails is the ActionMailbox
Conductor.  It offers a no-frills UI for sending email directly to your Rails
app. The two options for creating a new email are:

New inbound email by form

and 

New inbound email by source.


If you're not sure which one you need, you will most likely be able to have
everything you need with the form option.

The by source option is a bit more involved and takes an `.eml` file so much
more overhead but a great option for debugging and re-creating issues.

#### New Inbound Email Form

The new inbound email form has some options you're probably already familiar
with along with some fields to set additional mail headers, `X-Original-To` and
`In-Reply-To` (link what these do).

There's also the option to include an attachment to your inbound email (more on
that later).

Sending an email is the same process as submitting a form.  If you set up an
`:all` route in your ApplicationMailbox to a specific inbox.  Submitting this
form should deliver an email to that mailbox.

This process closely mimics the setup of ActionMailbox in real-life.

An email is sent to a specific email address
Your Email provider accepts and forwards the inbound email to your Rails app
Your Rails App creates an `ActtionMailbox::InboundEmail` record and updloads the email file to ActiveStorage until it can be processed. (pending -> processed state)
ActiveJob gets the email object and sends to the Mailbox.
The `InboundEmail` object takes the uploaded email file from ActiveStorate,
converts it to a string, creates a new `Mail` object (that's where the magic is)
The `process` method inside the mailbox is called on the mailbox where the email
is delivered too.


[Info Graphic Here]

There's a surprising amout of things happening in the background at first
glance.  I think this is a great way of hiding the complexitly, in a good way,
from everything that really happens with parsing the inbound mail so you
_really_ just have to focus on _what_ to do with the inbound email once you get
it.

#### Process inbound email in the Mailbox

If everything goes according to plan, your email will be sent to the `process`
method within a Mailbox.

This is where you're going to perform your business logic (or do the thing why
you created the mailbox for)

The ActionMailbox::InboundEmail is that object that's delivered to the `process`
method.  If you look at the generated migration above, there's not a lot to this
class on the surface.  It just has a status, some references and an
ActiveStorage attachment for the original email file sent.

```ruby
  class InboundEmail < Record
    self.table_name = "action_mailbox_inbound_emails"

    include Incineratable, MessageId, Routable

    has_one_attached :raw_email, service: ActionMailbox.storage_service
    enum status: %i[ pending processing delivered failed bounced ]

    def mail
      @mail ||= Mail.from_source(source)
    end

    def source
      @source ||= raw_email.download
    end

    def processed?
      delivered? || failed? || bounced?
    end
  end
end
```

The bulk of your processing of the inbound mail is most likely going to happen
with the Mail object.

[Ruby Mail](https://github.com/mikel/mail)

This is an invaluable resource for working with Email in Ruby.  Some of the
things you can do with ruby mail are fetching, `to`, `from`, `subject` and
others with ease.

```ruby
mail = Mail.new do
  from    'mikel@test.lindsaar.net'
  to      'you@test.lindsaar.net'
  subject 'This is a test email'
  body    File.read('body.txt')
end

mail.to_s #=> "From: mikel@test.lindsaar.net\r\nTo: you@...
```

It also allows us to dig deeper into things like headers, attachments, html and
text parts.

So what would this look like in practice?

Let's stick with the `RepliesMailbox` example from above. In this scenario, a
user gets a transactional email alert that someone replied to their post. We
include a reply email address with some unique identifier that let's us find
their reply.

EX:

`reply-123abc@example.org`


  <!-- def subscribe_info -->
  <!--   subscribe_info = mail.recipients.find { |r| MATCHER.match?(r) } -->
  <!--   return if subscribe_info.nil? -->
  <!--   user_name = subscribe_info[MATCHER, 1] -->
  <!--   list_slug = subscribe_info[MATCHER, 2] -->
  <!--   {user_name: user_name, list_slug: list_slug} -->
  <!-- end -->

  <!-- Think this was how I was doing the multi-match field stuff -->


<!-- `routing(/(.+)-(.+)@subscriptions.spotsquid.com/i => :subscriptions)` -->
`routing(/reply-(.+)@example.org/i => :subscriptions)`

For this specific example our routing in the ApplicationMailbox needs to be
updated a bit to break things up.

The above will return a MATCHER group (or something like that, maybe just one) that's an array
we can pull values from.

This value is going to be used to fetch the corresponding record.  It's up to
you how you would like to structure that.  In this case, it may make sense to
have the 123abc value map to the Post the reply belongs to?

```ruby
post_attribute = MATCHER.first
post = Post.find_by(post_attribute: post_attribute)
post.build_reply(body: mail.body)
```

In the above example, we're grabbing the value from the matcher and setting this
to `post_attribute`.  This will probably be something like a uuid, slug, or id
of your Record.

With that value from the email address, we're looking up the Post by that value.

Then we're building/creating a new reply that belongs to the Post and setting
the body of the reply to the email body.

There are a few other considerations to keep in mind when running this in
production (bounce_with, before_processing, raising errors, catchalls, etc.) but this is the basic flow of things.


#### Advanced Useage

- Before Processing
- bounce_with
- Parsing Attachments (save to ActiveStorage example - use pdfs as a good
  exmaple)
- Things to consider in production
- Setting up with one of the default ingress options.



#### Ingress Options

AWS is available and technically 'cheaper' if you don't value your time or your
sanity.

I use Postmark and I'll go through that example.
