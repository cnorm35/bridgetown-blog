---
layout: post
title:  Create PDF attachments from email with Action Mailbox
date:   2024-07-14 16:54:46 -0500
category: ruby
excerpt: "Create Active Storage attachments from attached files in emails."
author: cody
---

[Source Code](https://github.com/cnorm35/PdfMailboxExampleApp){:target="_blank"}

<!-- Uploading files and attachments is a common task in web applications. In this -->
<!-- post, we'll look at how to use Action Mailbox to create PDF attachments from an attachment in an inbound email. -->

<!-- We'll extract the PDF attachment from the inbound email and create a Active Storage -->
<!-- attachment from the attached file. -->

<!-- I'll also be demonstrating some ways to leverage the `before_processing` callback to confirm the -->
<!-- sender of the email matches a User record in our application then use the -->
<!-- `bounce_with` method to mark the email as bounced and notify the sender of the -->
<!-- bounce. -->

<!-- In addition to handling bounces, we'll also add some error handling to notify -->
<!-- the sender if the attachment is missing or not a PDF file. -->

<!-- From Video -->
Today, I’m going to show you how to save a PDF attachment from an inbound email using Action Mailbox a Ruby on Rails application.  I think this is a great example of a common feature that we can accomplish with email. This is going to allow your users to be able to perform tasks without even having to open your app.

 I’ll cover how to handle edge cases like checking for a valid user and handling missing or incorrect attachments.

This post will cover creating a new Rails app and adding Devise for a User model
to check against the sender of the email. Since this will be a new app, I'll also be installing Action Mailbox. Everything after that initial setup should apply to an existing app you're
trying to add this feature to.

If you'll be adding this feature to an existing app, you can skip over most of
that setup.


### Create your Rails app with Devise

Let's get started by creating a new Rails app and installing Devise for the User model.

```sh
$ rails new pdf_attachments_action_mailbox
```

Once the new app is created, move into the directory that was created and open
the project to add the Devise gem.

```ruby
# Gemfile
gem 'devise'
```

Once Devise is added to the `Gemfile` you can `bundle install` and run the
Devise install generators.

```sh
$ bundle install
$ rails generate devise:install
```

After running the Devise install generator, you'll see some output that mentions some changes to the views and
routes.  To keep things simple, I'll be skipping the User views and routes to
keep things focused on Action Mailbox.


Since we'll be sending _outbound_ emails to the sender to update them on the
status on the import, there is one change we need to add from
Devise.

Open the `config/environments/development.rb` file and add the following line
to the configuration:

```ruby
config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
```

With Devise installed, we can now generate the User model.

```sh
$ bin/rails generate devise User
$ bin/rails db:migrate
```

Since this app will be BYOV (Bring Your Own Views - I realize spelling it out
doesn't really save time but I think it's neat), we can create a `User` in
our Rails console to test with.

```ruby
irb> user = User.create(email: 'user@actionmailbox.pro', password: 'youvegotmail')
```


Next, we'll create a simple model to store the PDF attachments using Active Storage.

```sh
$ bin/rails g model ImportDocument user:references name:string
```

This creates a new model `ImportDocument` that belongs to a `User` and has a `name`
value.  `name` will store the subject of the email to make it easier to reference the import.

### Installing Action Mailbox

If you haven't already done so, you'll also need to install Action Mailbox.

```sh
$ bin/rails action_mailbox:install
```

One final change to make before running the migrations is to add the
`has_one_attached` method to the `ImportDocument` model.

```ruby
# app/models/import_document.rb
class ImportDocument < ApplicationRecord
  belongs_to :user
  has_one_attached :pdf
end
```

Now, running the migrations will create the tables for the `ImportDocument`
model and the tables needed for Action Mailbox, which includes Active Storage.

```sh
$ bin/rails db:migrate
```

This is all the setup we need to do to get started with the Action Mailbox side of things.  

Sidenote: I've found that keeping your mailboxes small and focused on a single task helps
keep things organized.  This mailbox will be responsible for saving the PDF
attachment from the email. If you decide to add different attachment types to
the `ImportDocument` model, you could create a new mailbox for each type.

To get started, we can create a new Mailbox to handle the inbound emails and save the PDF.

```sh
$ bin/rails generate mailbox Pdf
```

### Routing the email

Before processing the inbound email, we first need to route the email to the correct
mailbox.

```ruby
# app/mailboxes/application_mailbox.rb
class ApplicationMailbox
  # routing /pdf/i => :pdf
  routing all: :pdf
end
```

<!-- I'm including a commented out line that shows how you could route emails that -->
<!-- contain `pdf` in the to address of the inbound email.  In other words, sending -->
<!-- an inbound email to `pdf@inbound.yousite.com` or -->
<!-- `pdf-import@inbound,yoursite.com` would route the email to the pdf mailbox. -->

I've included a commented out line that gives a better idea of how you may want
to route this email outside of your development environment. This example would route emails that
contain `pdf` in the to address of the inbound email. In other words, sending an email to `pdf@inbound.yousite.com` or `pdf-import@inbound,yoursite.com` would route the email to the pdf mailbox.

When working on a new mailbox in development, I typically use the `all:` option
to route _any_ inbound email to the specified mailbox.  This rules out any
potential routing issues and lets you focus on the processing in the mailbox.

To confirm everything is working, we can send an email through the Rails Conductor and confirm it's been delivered.

With your Rails server running, open the Rails Conductor by visiting

`http://localhost:3000/rails/conductor/action_mailbox/inbound_emails`

and click the 'New inbound email by form' option or head there directly by going to

`http://localhost:3000/rails/conductor/action_mailbox/inbound_emails/new`

Since we're using the `all:` routing option, we don't have to worry about the
correct address to send the email to.  Just fill in the 'to' and 'from' fields and
submit the form.

<div class="my-5">
<img
    alt="Action Mailbox Rails Conductor"
    class="position-relative mx-auto rounded w-100 shadow-lg"
    src="/images/PdfRailsConductor.png"
    style="z-index: 10"
/>
</div>


After submitting the form, you'll land on the detail page for the `ActionMailbox::InboundEmail` record that was created from the form. Refresh the page and you should see the status change from `processing`
to `delivered`.  This means the email was successfully processed by the `PdfMailbox` and everything is working as expected.

<div class="my-5">
<img
    alt="Action Mailbox Rails Conductor Detail"
    class="position-relative mx-auto rounded w-100 shadow-lg"
    src="/images/DeliveredRailsConductor.png"
    style="z-index: 10"
/>
</div>

### Re-routing and Debugging

If something went wrong and the status didn't update to `delivered`, the
Rails conductor provides an easy button to re-route and deliver the email again.

I'm also going to add a `debugger` statement using [ruby debug](https://github.com/ruby/debug){:target="_blank"} to the `process` method of the `PdfMailbox` to show how you can inspect the inbound email and attachments or investigate any issues.

```ruby
# app/mailboxes/pdf_mailbox.rb
class PdfMailbox < ApplicationMailbox
  def process
    debugger
  end
end
```

After adding the `debugger` statement, restart your Rails server and click the 'Route again' button on the Inbound Email detail page.

Note: Since this app is only running with `rails server` and not using a Procfile, you'll see the debugger open in the terminal where the server is running.  If
you're using a Procfile and with something like [Overmind](https://github.com/DarthSim/overmind){:target="_blank"}, you can connect
to the debugger with `overmind connect web` or `overmind connect worker` depending on your specific setup.

<div class="my-5">
<img
    alt="Action Mailbox debugger in mailbox"
    class="position-relative mx-auto rounded w-100 shadow-lg"
    src="/images/MailboxDebugger.png"
    style="z-index: 10"
/>
</div>

In the above example, I'm interacting with the [Ruby Mail](https://github.com/mikel/mail){:target="_blank"} gem to parse
information from the inbound email. 

Since we're fully embracing the BYOV philosophy, you can also use the Rails console to
accomplish the same task.

Inside the Rails console, you can get the last `ActionMailbox::InboundEmail` record, update it's status to `pending`, then route the email to the `PdfMailbox`.

```ruby
# In the Rails Console
inbound_email = ActionMailbox::InboundEmail.last
inbound_email.pending!
inbound_email.route
```

If your `debugger` statement is still in the `process` method of the `PdfMailbox`, you'll see the debugger open in the terminal where the Rails console is running.

<div class="my-5">
<img
    alt="Action Mailbox debugger in Rails console"
    class="position-relative mx-auto rounded w-100 shadow-lg"
    src="/images/ConsoleMailboxDebugger.png"
    style="z-index: 10"
/>
</div>

### The Happy Path

To get things working, I'll first be focusing on the happy path.  This means
we'll just focus on creating the PDF attachment from the inbound email and
saving it to the `ImportDocument` model. In the later steps, we'll add some
error handling to handle edge cases and reply with emails alerting the sender of
the import status.

Before getting started, if you've not already done so, be sure to remove the
`debugger` statement from earlier.

Inside the `process` method of the `PdfMailbox`, we'll start be initializing a
new `ImportDocument` record and setting the `user` to the first User record.
For this to work, you'll need to have a User record in your database (snippet
for that above. After setting the `user`, we'll set the `name` to the subject of
the email so be sure to include a subject when sending the email with the Rails
Conductor.

```ruby
# app/mailboxes/pdf_mailbox.rb
  def process
    import_document = ImportDocument.new(
      user_id: User.first.id,
      name: mail.subject
    )
```

After initializing the `ImportDocument`, we'll attach the PDF file from the
inbound email to the `pdf` attribute of the `ImportDocument`.

```ruby
# app/mailboxes/pdf_mailbox.rb
  def process
    import_document = ImportDocument.new(
      user_id: User.first.id,
      name: mail.subject
    )

    import_document.pdf.attach(
      io: StringIO.new(mail.attachments.first.body.decoded),
      filename: mail.attachments.first.filename
    )
  end
```

This code is also assuming your inbound email has a PDF attachment. We'll worry
about the file type in a later step.

The last step in the happy path is to save the `ImportDocument` record.

```ruby
# app/mailboxes/pdf_mailbox.rb
  def process
    import_document = ImportDocument.new(
      user_id: User.first.id,
      name: mail.subject
    )

    import_document.pdf.attach(
      io: StringIO.new(mail.attachments.first.body.decoded),
      filename: mail.attachments.first.filename
    )
  end

  import_document.save!
```

Submitting the email through the Rails Conductor with all the required fields
should now create a new `ImportDocument` record with the PDF attachment from the
inbound email.


### Checking for an existing User

Anything live on the internet is only going to last a few days at best before it
starts getting inbound spam.  You may not want to try to create a new Import
Document from some spammers email signature.  To prevent this, we can check if a
User record exists for the sender of the email. If the User is found, we move on
to processing the email. If the User is not found, we'll mark the email as
bounced and send a reply to the sender.

Action Mailbox provides some methods to make this easy.  We'll use the `before_processing` callback to check for the User and the `bounce_with` method to mark the email as bounced. The `bounced_with` method take a mail object as an argument to send the bounce email to the sender.

```ruby
# app/mailboxes/pdf_mailbox.rb
class PdfMailbox < ApplicationMailbox
  before_processing :ensure_user

  def process
    import_document = ImportDocument.new(
      user_id: User.first.id,
      name: mail.subject
    )

    import_document.pdf.attach(
      io: StringIO.new(mail.attachments.first.body.decoded),
      filename: mail.attachments.first.filename
    )

    import_document.save!
  end

  private

  def ensure_user
    user = User.find_by(email: mail.from.first)
    unless user
        # bounce_with 
    end
  end
end
```

Inside the `ensure_user` method, we'll check if a User record exists for the email address of the sender. If a User is not found, we'll use the `bounce_with` method to mark the email as bounced and send a reply to the sender.

The `bounce_with` method requires a mailer object, we'll need to generate a new mailer to respond with.  There will be a few other types of emails used in the later steps so we'll create a new mailer for all of them at once.

### Generate Mailers to respond with


```sh
$ bin/rails generate mailer Pdf user_not_found missing_attachment bad_attachment_format import_complete
```

This will generate a new mailer with the methods `user_not_found`, `missing_attachment`, `bad_attachment_format`, and `import_complete`.

I won't be covering updating any of the mailer views, but you'll need to make a
couple of changes to each of the generated methods in the `PdfMailer`

```ruby
# app/mailers/pdf_mailer.rb
  def user_not_found
    @greeting = "User Not Found"
    @to = params[:to]

    mail to: @to
  end
```

Updating `@greeting` is optional, but you will to update the value passed to
`mail to:` to use the `to` value passed in the `params` hash. This is how we're
going to pass the email address of the sender to the mailer.

Bonus Step: Install `letter_opener_web` to view the _outbound_ emails easily in
your browser. There's more detailed information on that in the included video.

#### Wrapping up the User Not Found check

With our mailers created and updated, we can now pass that to the `bounce_with`
method.

```ruby
# app/mailboxes/pdf_mailbox.rb


def ensure_user
  user = User.find_by(email: mail.from.first)
  unless user
    bounce_with PdfMailer.with(to: mail.from.first).user_not_found
  end
end
```

`PdfMailer` is the mailer we generated earlier.  The `with` method is used to
pass params to the mailer.  `mail.from` is an array of email addresses that the
email was sent from.  We're using `mail.from.first` to get the first email
address (the same one we used to find the User record) and passing that as the
`to` value to the mailer.  The last part, `user_not_found`, is the mailer method
we're going to be sending.

If you re-start your Rails server and send an email from an email address that
does not match a User in your local database, you should see the email marked as
bounced and a reply sent to the sender.

<div class="my-5">
<img
    alt="Bounced Email in Rails Conductor"
    class="position-relative mx-auto rounded w-100 shadow-lg"
    src="/images/RailsConductorBounced.png"
    style="z-index: 10"
/>
</div>


Now, you're Action Mailbox app won't be trying to process documents and
attachments all willy-nilly.

Another one of the assumptions from the Happy Path is that the inbound email has
a PDF attachment.  In the next step, we'll add some error handling to check for
an attachment and reply to the sender if one is missing.

### Checking for an attachment

To check for an attachment before we attempt to process the email, we'll be
using another `before_processing` callback.  This time, we'll be checking if the
email has an attachment.  This portion won't be checking that it's a PDF file,
just that there is an attachment.  Letting the sender know they may have forgot
to attach the file is a better experience than getting a more generic error.

```ruby
# app/mailboxes/pdf_mailbox.rb
before_processing :ensure_attachment

  def ensure_attachment
    if mail.attachments.empty?
        # repy to sender
    end
  end
```

Next, we need to send a reply to the sender _and_ make sure we don't attempt to
process the email.  The main difference between this and the `bounce_with`
method is that the `bounce_with` updates the `status` of the
`ActionMailbox::InboundEmail` record to `bounced` instead of something like
`failed` or `delivered`.  In this situation, the sender was a valid User sending
to the correct email address so it's not really like it bounced.

If you followed the steps above for generating the mailers, you should have a `missing_attachment` method in the `PdfMailer` that we can use to reply to the sender.

```ruby
  def ensure_attachment
    if mail.attachments.empty?
      PdfMailer.with(to: mail.from.first).missing_attachment.deliver_later
    end
  end
```

This mailer is setup and called the same as the one passed to `bounce_with`
except for adding `deliver_later` to send the email asynchronously.

Our final step is to call `:abort` to stop the email from being processed.

```ruby
  def ensure_attachment
    if mail.attachments.empty?
      PdfMailer.with(to: mail.from.first).missing_attachment.deliver_later
      :abort
    end
  end
```

This ensures that we exit and don't attempt to process the email if there are no
attachments.

### Checking for a PDF attachment

Now we've confirmed the inbound email has at least one attachment, another
additional check we can add is to confirm the attachment is a PDF file.

```ruby
# app/mailboxes/pdf_mailbox.rb
  before_processing :ensure_attachment_format


  def ensure_attachment_format
    unless mail.attachments.first.content_type.start_with?("application/pdf")
      # send email and exit
    end
  end
```

For this approach, we'll be using another `before_processing` callback to check
the content type of the attachment.

Note:Outside of the scope of a tutorial, you may
want to combine or refactor some of the callbacks to suit your needs.  I wanted
to keep them separate to show easy ways we can add some incremental
improvements.

Inside our `ensure_attachment_format` method, we'll check if the content type of the first attachment starts with `application/pdf`.  If it does not, we'll send a reply to the sender and exit the processing of the email.

Now you know what to check for, we can re-use the same approach we used for the `ensure_attachment` callback to send a reply to the sender and exit the processing of the email.

```ruby
  def ensure_attachment_format
    unless mail.attachments.first.content_type.start_with?("application/pdf")
      PdfMailer.with(to: mail.from.first).bad_attachment_format.deliver_later
      :abort
    end
  end
```
### Reply on success

One last bit of polish we can add is to send a reply to the sender when the PDF
has been successfully saved. 

```ruby
# app/mailboxes/pdf_mailbox.rb
  def process
    import_document = ImportDocument.new(
      user_id: User.first.id,
      name: mail.subject
    )

    import_document.pdf.attach(
      io: StringIO.new(mail.attachments.first.body.decoded),
      filename: mail.attachments.first.filename
    )

    if import_document.save!
      PdfMailer.with(to: mail.from.first).import_complete.deliver_later
    end

  end
```

Now, if the `ImportDocument` is saved successfully, we'll send a reply to the
sender with the `import_complete` mailer method.

### Wrapping up

Within the context of the `PdfMailbox`, we've moved the common task of uploading
and processing files to email.  We've also added some error handling and UX
improvements to let the sender know if something went wrong or when it's
successful.

As an added perk, creating and processing the attachments via email with Action
Mailbox will perform the processing in the background whenever the
`ActionMailbox::InboundEmail` is processed.  This is a great way to handle large
files without tying up your web server.

The included video is a free preview from my upcoming course on Action Mailbox
where I'll be covering more advanced topics and edge cases.  If you'd like to
learn how to create more features like this one, the course is available for
pre-sale now.

Mailing List Form

[Action Mailbox Pro](https://store.codynorman.com/action-mailbox-pro)
