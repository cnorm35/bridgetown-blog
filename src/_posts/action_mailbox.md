---
layout: post
title:  In-Depth look at Action Mailbox
date:   2024-01-22 16:54:46 -0500
category: ruby
excerpt: "Email is a powerful and flexible way to extend the capabilities of your Rails application. It’s a familiar and low friction way for users to interact with your app."
author: cody
---

I wouldn't say I love email, more like I'm in love with the _idea of email_.

Allowing inbound email is a powerful and flexible way to extend the capabilities of your Rails
application.  It's a familiar and low friction way for users to interact with
your app.

I think leveraging email is a great example of focusing on solving problems for
your users instead of just building software.

Luckily, Rails has an unsung hero in Action Mailbox.  Action Mailbox gives you a
familiar Rails-y way to accept and process inbound emails for your app.

This post will walk through getting started with Action Mailbox, how to process
your inbound emails and things to watch out for along the way.

### Action Mailbox Background

The Rails guides for the [Action Mailbox Basics](https://guides.rubyonrails.org/action_mailbox_basics.html#what-is-action-mailbox-questionmark){:target="_blank"} do a pretty good job of explaining what Action Mailbox is.  Here's how they describe what Action Mailbox is:


<blockquote>
  <cite>
    Action Mailbox routes incoming emails to controller-like mailboxes for processing in Rails. It ships with ingresses for Mailgun, Mandrill, Postmark, and SendGrid. You can also handle inbound mails directly via the built-in Exim, Postfix, and Qmail ingresses.
  </cite>
</blockquote>

<blockquote>
  <cite>
The inbound emails are turned into `InboundEmail` records using Active Record and feature life cycle tracking, storage of the original email on cloud storage via Active Storage, and responsible data handling with on-by-default incineration.
  </cite>
</blockquote>

<blockquote>
  <cite>
These inbound emails are routed asynchronously using Active Job to one or several dedicated mailboxes, which are capable of interacting directly with the rest of your domain model.
  </cite>
</blockquote>

Two really important items to note are:

` storage of the original email on cloud storage via Active Storage...` and `These
inbound emails are routed asynchronously using Active Job...`

It's easy to gloss over those requirements and can cause some issues when
deploying to production. That will be covered in more detail and include my experiences deploying to production.

Here's a rough breakdown of how Action Mailbox processes your inbound emails.

<div class="my-5">
<img
    alt="Action Mailbox Processing Flow"
    class="position-relative mx-auto rounded w-100 shadow-lg"
    src="/images/ActionMailboxFlow.png"
    style="z-index: 10"
/>
</div>


### Getting Started
Getting started with Action Mailbox is a pretty familiar process

To install Action Mailbox run the following commands

```bash
$ bin/rails action_mailbox:install
$ bin/rails db:migrate
```

The install task creates an `ApplicationMailbox` and a new migration for the InboundEmails.

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

If you've not already added Active Storage to your app, you'll see the required
migrations for Active Storage as well.

The other file that was created is our `ApplicationMailbox`.  You can think of
the `ApplicationMailbox` as the Post Office for your inbound emails.  Mail is delivered, sorted then delivered to the correct mailbox.
This file is going to route the `InboundEmail` to a mailbox that inherits from
`ApplicationMailbox`.

```ruby
# app/mailboxes/application_mailbox.rb
class ApplicationMailbox < ActionMailbox::Base
  # routing /something/i => :somewhere
end
```

**Generating a new mailbox**

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
and a single method - `process`.

The `process` method is where the magic happens but first, we need to direct our
`InboundEmail` to the `SupportMailbox`.

When your inbound email is routed to the `SupportMailbox` the `process` method
will be called.  This is where you'll put your processing logic.

For that to happen, we need to tell our `ApplicationMailbox` how to route
emails to this mailbox.

The `ApplicationMailbox` has a commented out example to help get us started

```ruby
# routing /something/i => :somewhere
```

In this commented out example, anything inbound email with a `to` address that
matches on `something` will get routed to the `SomewhereMailbox`.

The routing uses the ruby regex matcher to route emails to a mailbox.

When getting started with Action Mailbox, instead of spending time thinking
about how you're going to route your emails, I've found it helpful to use the
`:all` option to direct _any_ inbound email to a specific mailbox.

`routing all: :support`

```ruby
# app/mailboxes/application_mailbox.rb
class ApplicationMailbox < ActionMailbox::Base
  routing all: :support
  # routing(/support\./i => :support)
end
```

The above example will route all inbound emails to the `SupportMailbox`. This lets you start focusing on the processing logic of the mailbox right away and
update the routing as your needs evolve.

The commented out example shows how you would route any inbound email with a `to` address
containing `support` to the `SupportMailbox`.


With everything setup to forward _any_ inbound email to the `SupportMailbox` you may be wondering...

**How do I send an email to my local Rails App?**

<!-- It's time to have your ticket punched by the Rails Conductor -->

I would like to introduce you to the aptly named Rails Conductor.

### Sending Emails with Rails Conductor
With your app running, you can access the Rails Conductor at the following URL

`http://localhost:3000/rails/conductor/action_mailbox/inbound_emails`

<div class="my-5">
<img
    alt="Rails Inbound Eamil Conductor"
    class="position-relative mx-auto rounded w-100 shadow-lg"
    src="/images/RailsConductorForm.png"
    style="z-index: 10"
/>
</div>

This is one of my favorite hidden gems (yeah I said it) of Rails.
It offers a no-frills UI for sending email directly to your app.  You also have
the option of creating and sending and email via the built in form or by
pasting the source code of an email.

Using the form option should provide you with everything you need. The by source option is a bit more involved and takes to source code from an `.eml` file.  This is a great option for debugging and re-creating issues from production but is typically more than you need when getting started.

**New Inbound Email Form**

The new inbound email form has some options you're probably familiar with along
with the option to include an attachment to your inbound email.

There's also some additional fields for setting email headers that may be useful
in your processing of the inbound emails.

**X-Original-To** - The email address listed here is the original recipient of the email that was received.

**In-Reply-To** - are header values taken from the Message-ID header of the original


If you set up an `:all` route in your `ApplicationMailbox` to a specific inbox like the `SupportMailbox` example, submitting this form should deliver an email to that mailbox. After submitting your email, it will show as `pending` until Active Job sends the original inbound email to your Mailbox and processes it.  If that's successful, the `InboundEmail` status will be updated to `delivered`  and at some point will be incinerated.

Now we're sending email, let's recap what's happening:

- An email is sent to a specific email address.
- Your Email provider accepts and forwards the inbound email to your Rails app.
- Your Rails App creates an `ActtionMailbox::InboundEmail` record and uploads the email file to Active Storage until it can be processed.
- Active Job gets the email object and sends to the Mailbox.
- The `InboundEmail` object takes the uploaded email file from Active Storage, converts it to a string and creates a new `Mail` object.
- The `process` method inside the mailbox is called on the mailbox where the email
is delivered too.
- The `InboundEmail` is marked for incineration (record and raw_email attachment
  deleted).

There's a surprising amount of things happening in the background at first
glace.  I think this is a great example of how Rails can handle so much for you
allowing you to focus on the important things without setting up a ton of
boilerplate on your own.

### Process inbound email in the Mailbox

We know that the `process` method is going to be called withing the mailbox the
inbound email is routed.

The `ActionMailbox::InboundEmail` is that object that's delivered to the `process`
method.  If you look at the generated migration above, there's not a lot to this
class on the surface.  It has a status, some references and an
Active Storage attachment for the original email file sent `raw_email`.

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

- **pending** - Email has been received and uploaded to Active Storage.
- **processing** - Original email is being downloaded from Active Storage and
  sent to the appropriate mailbox.
- **delivered** - Email was successfully delivered to the mailbox with no
  exceptions or bounces.
- **failed** - An exception was raised while the inbound email was being processed.
- **bounced** - Mark the email as bounced using `boune_with`

Most of the information you'll need to process the inbound email is
provided by the `Mail` object which is available with the `mail` method within the
mailbox.

The `InboundEmail` class defines a `mail` method that downloads the original
email file from Active Storage and creates a new `Mail` object.

[Ruby Mail](https://github.com/mikel/mail){:target="_blank"} is an invaluable resource for working with email in Ruby.

```ruby
mail = Mail.new do
  from    'mikel@test.lindsaar.net'
  to      'you@test.lindsaar.net'
  subject 'This is a test email'
  body    File.read('body.txt')
end

mail.to_s #=> "From: mikel@test.lindsaar.net\r\nTo: you@...
```

Ruby Mail also allows us to dig deeper into things like headers, attachments, html and
text parts.

What does 'processing' the email actually look like?

Obviously, each case will be different but let's take a look at an example for parsing the body of the
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

In the example above, we're creating a new `SupportTicket` record from the information contained in the inbound email.

We're pulling the `mail.from`, `mail.subject`, and `mail.body` data provided by
the `Mail` gem and using that to create our new record.

Parsing inbound email allows you to collect the same information as if you directed your user to
a form to submit a new support request. You have all the information to create a new `SupportTicket` and your user had a more streamlined and familiar process.

The outbound mailer that gets sent as a reply can also contain some type of
unique identifier we can easily use to look up the original `SupportTicket` and
link new responses to the conversation.

Let's say our `SupportMailbox` has a UUID field on the model, we can include that
value in the email address, parse from the reply and link new responses to the
original `SupportTicket` and not create a new support ticket each time there's a
reply.

```ruby
class ApplicationMailbox < ActionMailbox::Base
  routing all: :support
  # routing(/support\./i => :support)
  routing(/support-(.+)@inbound.yoursite.com/i => :support_reply) -->
end
```

<div class="my-5">
<img
    alt="Rubular Regex Example"
    class="position-relative mx-auto rounded w-100 shadow-lg"
    src="/images/Rubular.png"
    style="z-index: 10"
/>
</div>

Testing out hte commented out match statement shows we get a match for our test
string and we have a result in our capture group

The `(.+)` is going to return a match group which will be that UUID included in the
email address.

That should give you all you need to target the original `SupportTicket` you
created and respond, escalate, close, etc.

That's the ruby way to link emails together but there are also some options
available in the Mail headers.

`In-Reply-to` contains a hash value of the original message if the subject email
message is a reply to another message.  This is how your mail clients thread
together your emails.

There are a few other considerations to keep in mind when running this in
production (bounce_with, before_processing, raising errors, catch all mailboxes, etc.) but this is the basic flow of things.


### Advanced Usage

As your logic and number of mailboxes continue to grow, there are some
more-advanced lesser-known options that may help you on your journey.

As you continue down the path of inbound email enlightenment, there are some
additional tools and methods that could make your journey easier.

**Callbacks**

Action Mailbox provides some callbacks that will be ran before, after or around
the `process` method.

Action Mailbox Callbacks [Source Code](https://github.com/rails/rails/blob/main/actionmailbox/lib/action_mailbox/callbacks.rb){:target="_blank"}

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
can find the User before doing any additional processing on the email.

Keeping your email delivery rates high and bounce rate low is an important part
of getting your emails to actually land in people's inbox.  Taking some additional
steps to ensure you at least _probably_ have a valid email address before doing
more work than you need to.


You may have noticed another new method in the code above.  `bounce_with`

`bounce_with` accepts a Mail message to send to the sender letting them know the
email bounced.  It also marks the `ActionMailbox::InboundEmail` status as
`bounced`.

[source code](https://github.com/rails/rails/blob/6b93fff8af32ef5e91f4ec3cfffb081d0553faf0/actionmailbox/lib/action_mailbox/base.rb#L105){:target="_blank"}

There's another similar method `bounced!`.  This one silently prevents any additional processing
without sending out an email alerting the sender of the bounce.


This stops the email from being processed and delivers an email (of your
choosing) to notify someone of the bounce (probably the sender)

The method accepts a mailer object (is that right?) along with any additional parameters the email might
require.

This might look something like this:

```ruby
bounce_with PostRequiredMailer.missing(inbound_email)
```


### Parsing Attachments

<!-- Parsing Attachments from E-mail.  As I've mentioned many times before, people -->
<!-- are going to be a lot more familiar with their email than the UI of your -->
<!-- application.  Allowing them to email attachments is a super familiar pattern and -->
<!-- something pretty much everyone knows how to do.  If they don't I'm honestly -->
<!-- quite impressed they managed to sign up for your app. -->

It's very likely people are exponentially more familiar with their chosen email
client than they are the UI of your application. Attaching a file to an email is
something they've probably done dozens if not hundreds of times.  Leverage this
familiarness by allowing attachments and documents to be sent and uploaded
through email.

Let's set I need some 'attachments' or files. For example, PDF documents from a User.  I
could give them the steps to walk through uploading everything on their own in
some sort of self-service portal, have them upload the required documents in a
forma...or I could lean on what they already know how to do.

In this example, We'll look at what the process would be for grabbing an
attached PDF from an email, creating an Active Storage object and alerting an
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
attributes.  Next, we're creating a new object and setting the `name` to the value of
the email subject and `email` to the first returned sender.

Don't forget the `mail` gem returns an array of string values when calling `from` so we grab
the first one.

Assuming our `LegacyDocument` has the Active Storage attachment set up correctly.
Remember, Active Storage is a requirement of Action Mailbox so we should already
have everything setup and only need to add the `has_one_attached
:imported_document` to the model.

```ruby
# app/models/LegacyDocument
has_one_attached :imported_document
```

This is how we can create a new Active Storage attachment of the PDF attachment
from the `InboundEmail`.


```ruby
    legacy_document.imported_document.attach(
      io: StringIO.new(mail.attachments.first.body.decoded),
      filename: mail.attachments.first.filename
    )
```


With our object created, this code is manually creating and attaching an
`ActiveStorage::Attachment` called `imported_document` on the `LegacyWaiver` model.

<!-- https://github.com/mikel/mail?tab=readme-ov-file#testing-and-extracting-attachments -->

`attachments` is a method from Ruby mail that returns a list of the attachments.

[Extracting Attachments](https://github.com/mikel/mail?tab=readme-ov-file#testing-and-extracting-attachments){:target="_blank"}

Aside from the `mail` methods for accessing data on the mail object, this code
is mostly Active Storage.  `mail.attachments.first.body.decoded` will return a
string representation of the attachment which we wrap in `StringIO` and send to
Active Storage.

Digging into Active Storage is out of scope for this post but the
general idea is you 'manually' attach the file (imported_document) to the parent object (`LegacyWaiver`)

Accepting attachments all willy-nilly like this does open yourself up to more
potential issues.  Some things you may want to consider are:  file size, file
type, dealing with multiple attachments with things like signatures, etc.

<!-- setting these limits and validations with a way to communicate the issue back to -->
<!-- the sender if something goes wrong can help with hard to track down failures. -->

### Ingress Options and Production Considerations

Default ingress options Action Mailbox provides are:

- Exim
- Mailgun
- Mandrill
- Postfix
- Postmark
- Qmail
- SendGrid.

More information for each option can be found in the [configuration section](https://guides.rubyonrails.org/action_mailbox_basics.html#configuration){:target="_blank"} of
the Rails Guides.

Postmark is my preferred email service provider from the default options Action
Mailbox provides. I have another articles with the detail for [deploying to
Postmark](/deploy_action_mailbox_with_postmark).

**Active Storage Service**

One thing I've found helpful with deploying Action Mailbox to production is to
bite the bullet up front and configure an external service like Amazon S3 for
Active Storage.  Using the disk option can work but if you're app is not running
in a single process, it can potentially cause some headaches.

Here's why.

When your inbound mail service (like Postmark) receives the inbound email, it
forwards it to your Rails application.

After the inbound email is received by your app, it _uploads the original email
file to Active Storage_

`ActionMailbox::InboundEmail` has an Active Storage attachment`raw_email` that is used for storing
the original inbound email file (`.eml`). If we look back at the `source` method, we see that
that file is downloaded from the Active Storage storage service with `raw_email.download`.

```ruby
# rails/actionmailbox/app/models/action_mailbox/inbound_email.rb
...

def source
  @source ||= raw_email.download
end
```

<!-- https://github.com/rails/rails/blob/6b93fff8af32ef5e91f4ec3cfffb081d0553faf0/actionmailbox/app/models/action_mailbox/inbound_email.rb#L40C19-L40C37 -->

This process is also handled asynchronously with Active Job or other job service you have
configured.

Let's say you have your Rails app deployed to somewhere like Heroku or Render.

On Heroku, you probably have 2 dynos.  One running your app, one running your
background processes. If you have your Active Storage service set to `Disk`, that
will store the inbound email on your app dyno.

When the worker dyno attempts to process the inbound email by downloading
the original attachment, it will look on the dyno it's being executed on.

However, the _actual_ inbound email is stored on a different dyno.

This means the `InboundEmail` will never be able to be processed because the
dyno running that code can't find the original attachment (it's on the
other server)

This means that the `InboundEmail` record will raise an exception when trying to
process the email since it can't find the original attachment (it's stored in
the dyno running the app)

Setting up S3 from the get-go can eliminate a lot of these headaches. It also
has the advantage of making it easier to grab the original email file for
debugging.  This is a great example of where the 'create email from source' in
the Rails Conductor comes in handy.

Rob Zolkos has a great write up on how you can do that [here](https://world.hey.com/robzolkos/debugging-production-actionmailbox-issues-in-development-f5886579){:target="_blank"}.

Thanks Rob!

### Using a subdomain for inbound mail

Another thing that I've found helpful when setting up Action Mailbox for
production is running all your inbound emails through a subdomain.

`inbound.yourapp.com`

This also helps keeping your MX records separate on DNS.  If you have internal
email addresses for you app with something like `name@yourapp.com`  and try to
accept inbound emails for your app at `reply-123@yourapp.com` it's pretty easy
to accidentally overwrite your current DNS settings for existing email stuff.

<!-- Ask me how I know... -->
I've learnt this lesson first hand...

Sometimes, mining for gems takes some diggin'.

Action Mailbox is a wonderful example of clean and concise code handling a
complex problem and have learned a lot reviewing the source code.

I think Action Mailbox is a vastly underrated features of rails and can add
powerful functionality in a way that feels very familiar to the standard Rails
Controllers.

Allowing your users to perform actions from inbound emails is a great way to add convenience and
flexibility to your Rails app.  I hope this article has demystified the process
and given you some ideas on how you can use inbound email in you Rails app.

Let me know if you have any questions, or spot any issues that should be
updated.  Keep your eye out for some more ActionMailbox content coming soon!
