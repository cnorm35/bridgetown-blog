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


<!-- I've said it before and I'll say it again... -->

<!-- I love building email and sms features into my apps. -->

<!-- These are two channels that are comfortable and familar to your users with low -->
<!-- friction. --> 

<!-- This article is going to focus on the email portion of this. Specifically, using -->
<!-- ActionMailbox to allow incoming emails to your Rails app. -->





<!-- MAKE Action Mailbox consistent (try to use ActionMailbox when referring to the -->
<!-- code) -->

I wouldn't say I love email, more like I'm in love with the _idea of email_.

Allowing inbound email is a powerful and flexible way to extend the capabilities of your Rails
application.  It's a familiar and low friction way for users to interact with
your app.

I think leveraging email is a great example of focusing on solving problems for
users instead of just building software.

Luckily, Rails has an unsung hero with ActionMailbox.  ActionMailbox gives you a
familiar Rails-y way to accept and proccess inbound emails for your app.

<!-- This post is going to walk through getting started with ActionMailbox, things to -->
<!-- consider on production and my experiences. -->

This post is going to walk through getting started with ActionMailbox and some
things to watch out for along the way.

<!-- ### What is Action Mailbox? -->
### ActionMailbox Background

Let's face it, if you're like me, and you see some familar looking commands to
get started at the top of a guide, I can't resist running it just to see what it
does.

<div class="tenor-gif-embed" data-postid="10193348" data-share-method="host" data-aspect-ratio="1" data-width="100%"><a href="https://tenor.com/view/still-gonna-send-it-gif-10193348">Still Gonna Send It GIF</a>from <a href="https://tenor.com/search/still+gonna+send+it-gifs">Still Gonna Send It GIFs</a></div> <script type="text/javascript" async src="https://tenor.com/embed.js"></script>

If you're going through the Action Mailbox basics Rails guides, you could skip
over some important peices of information.

The Rails guides for the [Action Mailbox Basics](https://guides.rubyonrails.org/action_mailbox_basics.html#what-is-action-mailbox-questionmark) do a pretty good job of explaining what ActionMailbox is.  Here's how they describe what Action Mailbox is:


<blockquote>
  <cite>
    Action Mailbox routes incoming emails to controller-like mailboxes for processing in Rails. It ships with ingresses for Mailgun, Mandrill, Postmark, and SendGrid. You can also handle inbound mails directly via the built-in Exim, Postfix, and Qmail ingresses.
  </cite>
</blockquote>

<blockquote>
  <cite>
The inbound emails are turned into InboundEmail records using Active Record and feature lifecycle tracking, storage of the original email on cloud storage via Active Storage, and responsible data handling with on-by-default incineration.
  </cite>
</blockquote>

<blockquote>
  <cite>
These inbound emails are routed asynchronously using Active Job to one or several dedicated mailboxes, which are capable of interacting directly with the rest of your domain model.
  </cite>
</blockquote>

<!-- Breaking things down, ActionMailbox gives us an `InboundEmail` Active Record object. -->

Two really important items to note are:

` storage of the original email on cloud storage via Active Storage...` and `These
inbound emails are routed asynchronously using Active Job...`

It's really easy to gloss over those requirements and can cause some issues when
deploying to production. I'll go into that in more detail when going over some
of my experiences deploying to production.

### Getting Started
Getting started with Action Mailbox is a pretty familiar process

To install ActionMailbox run the following commands

```bash
$ bin/rails action_mailbox:install
$ bin/rails db:migrate
```

The install task creates a ApplicationMailbox and a new migration for the InboundEmails.

```
 bin/rails action_mailbox:install
Copying application_mailbox.rb to app/mailboxes
      create  app/mailboxes/application_mailbox.rb
       rails  railties:install:migrations FROM=active_storage,action_mailbox
Copied migration 20240123213529_create_action_mailbox_tables.action_mailbox.rb from action_mailbox
```

The migration for the `InboundEmail` should look something like this

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

If you've not already added ActiveStorage to your app, you'll see the required
migrations for ActiveStorage as well.

The other file that was created is our `ApplicationMailbox`.  You can think of
the `ApplicationMailbox` like the Post Office.  Mail is delivered there, then
sorted and delivered to the correct mailbox. This file is going
to route the InboundEmails to a mailbox that inherits from
`ApplicationMailbox`

```ruby
# app/mailboxes/application_mailbox.rb
class ApplicationMailbox < ActionMailbox::Base
  # routing /something/i => :somewhere
end
```

<!-- Running the migrate task and restarting your app and just like that you're able -->
<!-- to accept inbound emails to your app.  Sort of. -->

<!-- The `ApplicationMailbox` is where all of your inbound emails will be sent.  This -->
<!-- file decides which mailbox to route the email to.  Like any good Rails tool, -->
<!-- there's a task to generate a new mailbox. -->

Like any good Rails tool, there's a task to generate a new mailbox.

```
bin/rails generate mailbox support
```

The above command will create a new mailbox that looks something like this.

```ruby
class SupportMailbox < ApplicationMailbox
  def process
  end
end
```

As mentioned earlier, our `SupportMailbox` inherits from `ApplicationMailbox`
and has one method `process`

The `process` method is where the magic happens but first, we need to direct our
InboundEmail to the SupportMailbox.

When your inbound email is routed to the `SupportMailbox` the `process` method
will be called.  This is where you'll put your processing logic.

But for that to happen, we need to tell our `ApplicationMailbox` how to route
emails to this mailbox.

<!-- There are a couple of options for directing the InboundEmail to the correct -->
<!-- mailbox.  What you'll be using in production (most likely) will be matching -->
<!-- emails that include `@replies` in the correct portion of the email address. -->

The `ApplicationMailbox` has a commented out example to help get us started

` # routing /something/i => :somewhere`

In this commented out example, anything inbound email with a `to` address that
matches on `something` will get routed to the `SomewhereMailbox`.

The routing uses the ruby regex matcher to route emails to a mailbox.



`routing(/(.+)-(.+)@subscriptions.spotsquid.com/i => :subscriptions)`

When getting started with Action Mailbox, instead of spending time thinking
about how you're going to route your emails, I've found it helpful to use the
`:all` option to direct _any_ inbound email to a specific mailbox.

<!-- For development purposes, especially when you have a single Mailbox, I've found -->
<!-- it helpful to use `:all` to send _any_ inbound email to the mailbox you pass it. -->

`routing all: :support`

```ruby
# app/mailboxes/application_mailbox.rb
class ApplicationMailbox < ActionMailbox::Base
  routing all: :support
  # routing(/support\./i => :support)
end
```

This will route all inbound emails to the `SupportMailbox`.  The commented out
example shows how you would route any inbound email with a `to` address
containing `support` to the `SupportMailbox`.

This lets you start focuing on the business logic of the mailbox right away and
update the routing as your needs evolve.

Now, everything is setup to forward _any_ inbound email to the `SupportMailbox` you may be wondering...

**How do I send an email to my local Rails App?**

<!-- It's time to have your ticket punched by the Rails Conductor -->

I would like to introduce you to the aptly named Rails Conductor.

[SCREENSHOT]

### Sending Emails with Rails Conductor
With your app running, you can access the Rails Conductor at the following URL

http://localhost:3000/rails/conductor/action_mailbox/inbound_emails

<!-- This is one of my favorite hidden gems (yeah I said it) of Rails is the ActionMailbox -->
<!-- Conductor.  It offers a no-frills UI for sending email directly to your Rails -->
<!-- app. The two options for creating a new email are: -->

This is one of my favorite hidden gems (yeah I said it) of the Rails ecosystem.
It offers a no-frills UI for sending email directly to your app.  You also have
the option of creating and sending and email form the built in form or by
uploading the source code of the email.

<!-- New inbound email by form -->

<!-- and --> 

<!-- New inbound email by source. -->

<!-- If you're not sure which one you need, you'll most likely be able to have -->
<!-- everything you need with the form option. -->

<!-- Using the form option should provide you with everything you need.  The source -->
<!-- option is handy when you're trying to re-create issues on production by -->
<!-- downloading the email source (.eml file) and using it in your local environment. -->

Using the form option should provide you with everything you need. The by source option is a bit more involved and takes an `.eml` file, or at least the code contained within the file.  This is a great option for debugging and re-creating issues from production.

**New Inbound Email Form**

The new inbound email form has some options you're probably already familiar
with along with some fields to set additional mail headers, `X-Original-To` and
`In-Reply-To` (link what these do).

```

X-Original-To
    The email address listed here is the original recipient of the email that was received.
```

```
The In-Reply-To headers are taken from the Message-ID header of the original
```

There's also the option to include an attachment to your inbound email.

Sending an inbound email to your app is the same process as submitting a form.

If you set up an `:all` route in your ApplicationMailbox to a specific inbox like the `SupportMailbox` example, submitting this form should deliver an email to that mailbox. After submitting your email, it will show as `pending` until ActiveJob sends the original inbound email to your Mailbox and processes it.  If that's successful, the InboundEmail status will be updated to `delivered`  and at some point will be incenerated.

<!-- This process closely mimics the setup of ActionMailbox in real-life. -->

Now we're sending email, let's recap what's happening:

- An email is sent to a specific email address
- Your Email provider accepts and forwards the inbound email to your Rails app
- Your Rails App creates an `ActtionMailbox::InboundEmail` record and updloads the email file to ActiveStorage until it can be processed. (pending -> processed state)
- ActiveJob gets the email object and sends to the Mailbox.
- The `InboundEmail` object takes the uploaded email file from ActiveStorage
converts it to a string, creates a new `Mail` object (that's where the magic is)
- The `process` method inside the mailbox is called on the mailbox where the email
is delivered too.
- The `InboundEmail` is marked for deletion (default 30 days?)


[Info Graphic Here]

<!-- There's a surprising amout of things happening in the background at first -->
<!-- glance.  I think this is a great way of hiding the complexitly, in a good way, -->
<!-- from everything that really happens with parsing the inbound mail so you -->
<!-- _really_ just have to focus on _what_ to do with the inbound email once you get -->
<!-- it. -->

There's a surprising amount of things happening in the background at first
glace.  I think this is a great example of how Rails can handle so much for you
allowing you to focus on the important things without setting up a ton of
boilerplate on your own.

#### Process inbound email in the Mailbox

<!-- If everything goes according to plan, your email will be sent to the `process` -->
<!-- method within a Mailbox. -->

<!-- This is where you're going to perform your business logic (or do the thing why -->
<!-- you created the mailbox for) -->

We know that the `process` method is going to be called withing the mailbox the
inbound email is routed.

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

The different statues are:

- **pending** - email has been received and uploaded to ActiveStorage
- **processing** - the original email is being downloaded from ActiveStorage and
  sent to the approiate mailbox.
- **delivered** - the email was successfully delivered to the mailbox with no
  exceptions or bounces.
- **failed** - an exception was raised while the inbound email was being processed.
- **bounced** - Mark the email as bounced using `boune_with`

The bulk of your processing of the inbound mail is most likely going to happen
with the Mail object.

Most of the information you'll need to actually process the inbound email is
provided by the Mail object which is available with the `mail` method within the
mailbox.

The `InboundEmail` class defines a `mail` method that downloads the original
email file from ActiveStorage and creates a new `Mail` object

This is an invaluable resource for working with Email in Ruby.  Some of the
things you can do with ruby mail are fetching, `to`, `from`, `subject` and
others with ease.

[Ruby Mail](https://github.com/mikel/mail)


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


<!-- So what would this look like in practice? -->

<!-- Let's stick with the `SupportMailbox` example from above. In this scenario, a -->
<!-- user gets a transactional email alert that someone replied to their post. We -->
<!-- include a reply email address with some unique identifier that let's us find -->
<!-- their reply. -->
What does 'processing' the email actually look like?  Obviously, each case will
be different but let's take a look at an example for parsing the body of the
email and creating a new support ticket.


```ruby
class SupportMailbox < ApplicationMailbox
  def process
    support_ticket = SupportTicket.create(from_email: mail.from,
                      subject: mail.subject,
                      body: mail.body)
    SupportMailer.new_ticket(support_ticket).deliver_now
  end
end
```

In the above example, we're grabbing the value from the matcher and setting this
to `post_attribute`.  This will probably be something like a uuid, slug, or id
of your Record.

In the example above, the basic idea of what's happening is we're creating a new
`SupportTicket` record from the infomation contained in the inbound email.

We're pulling the `mail.from`, `mail.subject`, and `mail.body` data provided by
the `Mail` gem and using that to create our new record.

This allows you to collect the same information as if you directed your user to
a form to submit a new support request.  Now you have all the information to
create a new SupportTicket and your user had a more streamlined process.

The outbound mailer that gets sent as a reply can also contain some type of
unique identifier we can easily use to look up the original SupportTicket and
link new responsese to the conversation.

Let's say our SupportMailbox has a UUID field on the model, we can include that
value in the email address, parse from the reply and link new responses to the
original SupportTicket and not create a new support ticket each time there's a
reply.

```ruby
class ApplicationMailbox < ActionMailbox::Base
  routing all: :support
  # routing(/support\./i => :support)
  routing(/support-(.+)@inbound.yoursite.com/i => :support_reply) -->
end
```

[RUBULAR SCREENSHOT]

The `(.+)` is going to return a matcher which will be that UUID included in the
email address.

That should give you all you need to target the original SupportTicket you
created and respond, esclate, close, etc.

That's the ruby way to link emails together but there are also some options
available in the Mail headers.

`In-Reply-to` contains a hash value of the original message if the subject email
message is a reply to another message.  This is how your mail clients thread
together your emails.

There are a few other considerations to keep in mind when running this in
production (bounce_with, before_processing, raising errors, catch all mailboxes, etc.) but this is the basic flow of things.


#### Advanced Useage

As your logic and number of mailboxes continue to grow, there are some
more-advanced lesser-known options that may help you on your journey.

As you continue down the path of inbound email enlightenment, there are some
additional tools and methods that could make your journey easier.

**Callbacks**
Action Mailbox provides some callbacks that will be ran before, after or around
the `process` method.

Action Mailbox Callbacks [Source Code](https://github.com/rails/rails/blob/main/actionmailbox/lib/action_mailbox/callbacks.rb)


```ruby
# frozen_string_literal: true

require "active_support/callbacks"

module ActionMailbox
  # = Action Mailbox \Callbacks
  #
  # Defines the callbacks related to processing.
  module Callbacks
    extend  ActiveSupport::Concern
    include ActiveSupport::Callbacks

    TERMINATOR = ->(mailbox, chain) do
      chain.call
      mailbox.finished_processing?
    end

    included do
      define_callbacks :process, terminator: TERMINATOR, skip_after_callbacks_if_terminated: true
    end

    class_methods do
      def before_processing(*methods, &block)
        set_callback(:process, :before, *methods, &block)
      end

      def after_processing(*methods, &block)
        set_callback(:process, :after, *methods, &block)
      end

      def around_processing(*methods, &block)
        set_callback(:process, :around, *methods, &block)
      end
    end
  end
end
```

Like mentioned above, these callbacks relate to the `process` method.  If you
would like to perform some actions or ensure certain values are present the
`before_processing`, like confirming your can find a User record that matches
the email for the inbound message. In our `SupportTicket`
example above, sending the confirmation email after a new ticket is created is a
good example of something that would fit in the `after_processing` method.


```ruby

class SupportMailbox < ApplicationMailbox
  before_processing :ensure_user

  def process
    ### code for creating SupportTicket
  end

  private

  def from_email
    mail.from.first
    # mail.from returns Array
  end

  def ensure_user
    @user = User.find_by(email: from_email)
    unless @user
      bounce_with Mailer.post_not_found(mail)
    end
  end
end
```

Here's a basic example of using a `before_processing` callback to make sure we
can find the User before doing any additional proccessing on the email.

Keeping your email delivery rates high and bounce rate low is an important part
of getting your emails to actually land in people's inbox.  Taking some additional
steps to ensure you at least _probably_ have a valid email address before doing
more work than you need to.


You may have noticed another new method in the code above.  `bounce_with`

`bounce_with` accepts a Mail message to send to the sender letting them know the
email bounced.  It also marks the `ActionMailbox::InboundEmail` status as
`bounced`.

[source code](https://github.com/rails/rails/blob/6b93fff8af32ef5e91f4ec3cfffb081d0553faf0/actionmailbox/lib/action_mailbox/base.rb#L105)

There's another similar mehtod `bounced!`.  This one silently prevents any additional processing
without sending out an email alerting the sender of the bounce.


This stops the email from being processed and delivers an email (of your
choosing) to notify someone of the bounce (probably the sender)

The method accepts a mailer object (is that right?) along with any additionl params the email might
require.

This might look somehting like this:

```ruby
bounce_with PostRequiredMailer.missing(inbound_email)
```


#### Parsing Attachments

Parsing Attachments from E-mail.  As I've mentioned many times before, people
are going to be a lot more familiar with their email than the UI of your
application.  Allowing them to email attachments is a super familiar pattern and
something pretty much everyone knows how to do.  If they don't I'm honestly
quite impressed they managed to sign up for your app.

Let's set I need some 'attachments', in this case, PDF documents from a User.  I
could give them the steps to walk through uploading everything on their own in
some sort of self-service portal, or I could lean on what they already know how
to do.

In this example, We'll look at what the process would be for grabbing an
attached PDF from an email, creating an ActiveStorage object and alerting an
admin that a new document is ready.

```ruby
class LegacyDocumentMailbox < ApplicationMailbox
  def process
    create_legacy_document
  end

  def create_legacy_document

    legacy_document = LegacyDocument.new(
      name: mail.subject,
      email: mail.from.first,
    )

    # imported_document is the attachment name on LegacyDocument
    legacy_document.imported_document.attach(
      io: StringIO.new(mail.attachments.first.body.decoded),
      filename: mail.attachments.first.filename
    )

    legacy_document.save!
    # maybe up this to rescue or bounce with on fail?
  end
end

```

All of our business logic is within the `create_legacy_document` method, so
let's break down what's happening.

```ruby
legacy_document = LegacyDocument.new(
    name: mail.subject,
    email: mail.from.first,
)
```

We have a `LegacyDocument` active record object that has `name` and `email`
attributes.  We're creaing a new object and setting the `name` to the value of
the email subject and `email` to the first returned sender.

The `mail` gem returns an array of string values when calling `from` so we grab
the first one.

```ruby
    legacy_document.imported_document.attach(
      io: StringIO.new(mail.attachments.first.body.decoded),
      filename: mail.attachments.first.filename
    )
```

```ruby
# app/models/LegacyDocument
has_one_attached :imported_document
```

With our object created, this code is manually creating and attaching an
ActiveStorage::Attachment called `imported_document` on the LegacyWaiver model

https://github.com/mikel/mail?tab=readme-ov-file#testing-and-extracting-attachments

`attachments` is a method from Ruby mail that returns a list of the attachments.

Aside from the `mail` methods for accessing data on the mail object, this code
is mostly ActiveStorage.  `mail.attachments.first.body.decoded` will return a
string representation of the attachment which we wrap in StringIO and send to
ActiveStorage. Digging into ActiveStorage is out of scope for this post but the
general idea is you 'manually' attach the file (imported_document) to the parent object
(LegacyWaiver)

Accepting attachments all willy-nilly like this does open yourself up to more
potential issues.  Some things you may want to consider are:  file size, file
type, etc.

setting these limits and validations with a way to communicate the issue back to
the sender if something goes wrong can help with hard to track down failures.


#### Ingress Options and Production Considerations

Default ingress options are:

Exim, Mailgun, Mandrill, Postfix, Postmark, Qmail, SendGrid.

Info and more links on each on can be found in the guides https://guides.rubyonrails.org/action_mailbox_basics.html#configuration


You may notice, there's no default option for AWS/SES email. There is a third
party ingress https://github.com/bobf/action_mailbox_amazon_ingress  I tried it
at one point and no complaints about the ingress but setting up everything
required on the Amazon side was as-usual, a nightmare.

Speaking of Amazon, I'm not telling you how to live your life, but you
_probably_ want to set up S3 for ActiveStorage ahead of time.  We mentioned it
briefly at the start of the article but this is something than can cause some
headaches in production.

Here's why:

When your inbound mail service (like Postmark) receives the inbound email, it
forwards it to your Rails application.

After the inbound email is recieved by your app, it _uploads the original email
file to Active Storage_ 

ActionMailbox::InboundEmail has an attached `raw_email` that is used for storing
the original inbound email. If we look back at the `source` method, we see that
that file is downloaded from the ActiveStorage storage service.
`raw_email.download`


https://github.com/rails/rails/blob/6b93fff8af32ef5e91f4ec3cfffb081d0553faf0/actionmailbox/app/models/action_mailbox/inbound_email.rb#L40C19-L40C37

This process is also handled with ActiveJob or whatever job service you have
configured.

Let's say you have your Rails app deployed to somewhere like Heroku or Render.

On Heroku, you probably have 2 dynos.  One running your app, one running your
background processes. If you have your ActiveStorage service set to local, that
will store the inbound email on your web container.

When the worker container attempts to process the inbound email by downloading
the original attachemnt, it will look on the container it's being executed on.
However, the _actual_ inbound email is stored on a different container.

This means the InboundEmail will never be able to be processed becaue the
container running that code can't find the original attachment (it's on the
other server)

Setting up S3 from the get-go can eliminate a lot of these headaches. It also
has the advantage of making it easier to grab the original email file for
debuggng.  This is a great example of where the 'create email from source' in
the Rails conductor comes in super handy.

Rob Zolkos has a great write up on how you can do that [here](https://world.hey.com/robzolkos/debugging-production-actionmailbox-issues-in-development-f5886579)

Thanks Rob!

Another thing that I've found helpful when setting up ActionMailbox for
production is running all your inbound emails through a subdomain.

`inbound.yourapp.com`

What you name your sub-domain isn't super important but it helps with making the
matchers easier in the ApplicationMailbox since you can assume anything ending
in `inbound.yourapp.com` needs to go _somewhere_.

It also helps keeping your MX records separate on DNS.  If you have internal
email addresses for you app with something like `name@yourapp.com`  and try to
accept inbound emails for your app at `reply-123@yourapp.com` it's pretty easy
to accidently overwrite your current DNS settings for existing email stuff.

Ask me how I know.


I usually go with Postmark and will be posting something soon on getting
ActionMailbox running live with Postmark.

I think ActionMailbox is a vastly underrated features of rails and can add
powerful functionality in a way that feels very Rails-y.

Let me know if you have any questions, keep your eye out for some more
ActionMailbox stuff.

