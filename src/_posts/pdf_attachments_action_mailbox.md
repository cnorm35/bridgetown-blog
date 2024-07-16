---
layout: post
title:  Create PDF attachments from email with Action Mailbox
date:   2024-07-14 16:54:46 -0500
category: ruby
excerpt: "Create Active Storage attachments from attached files in emails."
author: cody
---

Uploading files is a common task you'll encounter in web applications. In this
post, we'll look at how to create PDF attachments from files attached to
incoming emails using Action Mailbox in Rails.

We'll extract the PDF attachment from the inbound email and create a Active Storage
attachment from the attached file.


We'll take a look at using the `before_processing` callback to confirm the
sender of the email matches a User record in our application and use the
`bounce_with` method to mark the email as bounced and notify the sender of the
bounce.

<!-- We'll also add some error handling to notify the sender if the attachment is -->
<!-- missing or not a PDF file. -->
In addition to handling bounces, we'll also add some error handling to notify
the sender if the attachment is missing or not a PDF file.



From Video
Today, I’m going to show you how to save a PDF attachment from an inbound email using Action Mailbox.  This is a great example of a common task in an application that we can accomplish with email. This is going to allow your users to be able to perform tasks without even having to open your app.

 I’ll cover how to handle various edge cases, like checking for a valid user and handling missing or incorrect attachments.

This post will cover creating a new Rails app and adding Devise for a User model
to check against the sender of the email. I'll also be installing Action Mailbox
but everything after that initial setup should apply to an existing app you're
trying to add this feature to.


Let's get started by creating a new Rails app with a User model and Devise installed.

```sh
$ rails new pdf_attachments_action_mailbox
```
After the new app is created, we'll add Devise to the Gemfile and run the Devise generator to create a User model.

```ruby
# Gemfile
gem 'devise'
```

```sh
$ bundle install
$ rails generate devise:install
```

You'll see some output from the Devise generator that mentions the views and
routes.  To keep things simple, I'll be skipping the User views and routes to
keep things focused on Action Mailbox. 


Since we'll be sending outbound emails to the sender to update them on the
status on the import, we do need to add one of the suggested changes from
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
doesn't really save any time but I like the acronym), we can create a User in
our Rails console to test with.

```ruby
irb> user = User.create(email: 'user@actionmailbox.pro', password: 'youvegotmail')
```


Next, we'll create a simple model to store the PDF attachments.

```sh
$ bin/rails g model ImportDocument user:references name:string
```

This creates a new model `ImportDocument` that belongs to a User and has a name
value.  `name` will store the subject of the email to help reference the import
a little easier.

If you haven't already, you'll also need to install Action Mailbox.

```sh
$ bin/rails action_mailbox:install
```

One final change to make before running our migrations to to add the
`has_one_attached` method to the `ImportDocument` model.

```ruby
# app/models/import_document.rb
class ImportDocument < ApplicationRecord
  belongs_to :user
  has_one_attached :pdf
end
```

Now, running the migrations will create the tables for the `ImportDocument`
model and the tables needed for Action Mailbox, which includes Active Storage

```sh
$ bin/rails db:migrate
```

This is all the setup we need to do to get started with Action Mailbox.  Next,
we'll create a new Mailbox to handle the inbound emails and save the PDF.

```sh
$ bin/rails generate mailbox Pdf
```

I've found that keeping your mailboxes small and focused on a single task helps
keep things organized.  This mailbox will be responsible for saving the PDF
attachment from the email. If you decide to add different attachment types to
the `ImportDocument` model, you could create a new mailbox for each type.

Before processing the inbound email, we need to route the email to the correct
mailbox.  

```ruby
# app/mailboxes/application_mailbox.rb
class ApplicationMailbox
  # routing /pdf/i => :pdf
  routing all: :pdf
end
```

I'm including a commented out line that shows how you could route emails that
contain `pdf` in the `to` address of the inbound email.  In otherwords, sending
an inbound email to `pdf@inbound.yousite.con` would route the email to the pdf
mailbox.

When working on a new mailbox in development, I typically use the `all:` option
to route _any_ inbound email to the specified mailbox.  This rules out any
potential routing issues and lets you focus on the processing in the mailbox.

Notes on next steps:
Send example inbound email in Rails conductor, show that it’s been delivered.

Add `debugger` to `PdfMailbox`

restart server

click the ‘Route Again’ button and show how the debugger is called and you can inspect stuff.

In the Rails console show how you can do that manually

```json
inbound_email = ActionMailbox::InboundEmail.last
inbound_email.pending!
inbound_email.route

(opens debugger)
```
