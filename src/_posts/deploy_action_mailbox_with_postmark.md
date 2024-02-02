---
layout: post
title:  Inbox Hero
date:   2024-01-23 16:54:46 -0500
category: ruby
excerpt: "Deploying Action Mailbox to Postmark"
author: cody
---

ref post: https://postmarkapp.com/blog/handling-inbound-emails-in-rails-using-postmark 


In another post, I went into some of the detail aroind ActionMailbox and some
ways it can be used.

I wanted to have a separate post that was just for deploying your app with
ActionMailbox to production somewhere.

The current options Rails offers are:

Exim, Mailgun, Mandrill, Postfix, Postmark, Qmail, and SendGrid.

I've used Postmark quite a bit over the years for transactional email and have
always had a good experience so this is my default provider and the one we're
going to walk through in this post.

Before we get started, there are a couple of things you'll need that will help
your setup go smoother.

A Postmark Account
A domain to use for the mailer.

(can probably even leave out the domain stuff and maybe just use ngrok? not sure
about the approval stuff)

This post will assume your app is already deployed somewhere and won't be
covering deploying your Rails app, just connecting Postmark for your inbound
emails.  If you're looking for some help deploying your app, I've been a long
time user of [Hatchbox](https://hatchbox.io/) and can't recommend them enough.

(Maybe have something that's using ngrok to send the inbound mail to local app?)
```
  # use ngrok and update postmark settings to test locally
  # config.action_mailbox.ingress = :postmark
```

With your ActionMailbox app ready to go, let's start adding some of the configs
needed for Postmark.

[Rails Guide -
Postmark](https://guides.rubyonrails.org/action_mailbox_basics.html#postmark)

The first thing we need to is set our ingress for ActionMailbox in
`config/environments/production.rb`

```ruby
# config/environments/production.rb
config.action_mailbox.ingress = :postmark
```

After creating a Postmark account, the next thing you'll need do is to create a
new 'Server'.

In Postmark, a server is a way to organize or group incoming or outgoing email.
You can read more about Postmark servers [here](https://postmarkapp.com/developer/user-guide/managing-your-account/managing-servers)

[POSTMARK SERVER SETUP SCREENSHOT]

After creating the server, you'll see 3 different 'Streams' listed on the
dashboard.

Postmark Streams Info:
> Message Streams is how we separate different types of mail to ensure the highest deliverability for each. Pick the one that’s right for your email:

    Transactional streams are for sending time-sensitive messages triggered for one recipient at a time.
    Broadcasts are for messages sent to many recipients at once, like marketing campaigns or newsletters.
    Inbound stream is a way for your application to receive email.


For configuring our ActionMailbox we'll be using the one that says 'Default
Inbound Stream'

After first clicking on the Default Inbound Stream link, you should land on the
'Setup Instructions' page.  Right at the top, you'll see the option to get your
servers inbound email address.  

Setting up production email is fraught with peril.  Well, maybe that's being a
bit dramatic but it can still be a bumpy ride.  Debugging and diagnosing
production issues with email can feel like trying to look inside a black box
that has another black box inside.  There's lot of small things that can go
wrong.  For this reason, in order to preserve my time and sanity as best as
possible, I try to do a series of small checks along the way to make sure things
are working as expected.


When digging into production issues, ruling out possible issues is a big part of
that.


Just to make sure out Inbound Stream has been set up correctly and ready for
use, we can fire off a quick email to the server's inbound email address at the
top-section.  It should look something like this

`123456e8b05e594a3a18129f293281469@inbound.postmarkapp.com`

After sending your inbound email to the server, clicking on the 'Activity' tab
for your inbound server should show the inbound email you just sent.

[SCREENSHOT]

<!-- Now we know that things should be configured correctly on the Postmark side and -->
<!-- can move on to the next step. -->


<!-- With the inbound server configured, the next thing we need to do is get the API -->
<!-- key for our server. -->

<!-- Remember that the Server is the grouping of the Message Streams (the Inbound one -->
<!-- we're using) so navigate back to the server and click on 'API Tokens' -->

<!-- [SCREENSHOT] -->


<!-- ^ do I even need the API key?  I think I just need my password (weird I know) -->

<!-- (Think I can skip the API token stuff) -->


Now we need to create a secure password to authenticate our webhook requests
from Postmark.

In your Rails console, we can use SecureRandom to generate a password for us.



```ruby
SecureRandom.alphanumeric
=> "12312ddfgdfg"
```

You'll need to copy the output of that command and add it to our encrypted
credentials.

Running `bin/rails credentials:edit` (find that vs code hack thing) will open
the main encrypted creds file. All the credentials in this file will apply to
_all_ environments unless we have a specifc env file.

In other words, these credentials are going to apply to our `development` and
`production` environment unless I created a `credentials/development.yml` (or
whatever that is) file.  Eventually you'll probably want to separate that out
with different settings for different environments.  However, making the
password available in our `development` environment is going to make it easier
connecting this to our local environment with ngrok (or maybe that cloudflare
one)

```ruby
action_mailbox:
  ingress_password: 12312ddfgdfg
```

I'm a huge fan of using Rails encrypted credentials but if you'd like to stick
with using environment variables, you can set that password to
`RAILS_INBOUND_EMAIL_PASSWORD`

Speaking of checking things along the way, now would be a good time to make sure
these changes are _deployed_ and checked to make sure the values are set.

A quick visit to our old friend the Rails console is all we need to confirm our
password has been set.


```ruby
Rails.application.credentials.action_mailbox
=> {ingress_password: 12312ddfgdfg}


```

Or if you set the value with an environment variable

```ruby
ENV["RAILS_INBOUND_EMAIL_PASSWORD"]
=> 12312ddfgdfg
```

Anytime I have issues configuring a new service, the credentials are the first
place I check so this is a great checkpoint in the process.


With the password on the server, we need to setup an inbound webhook for
Postmark.  This inbound webhook is going to receive the incoming email and send
the data to our postmark inbound email controller ActionMailbox provides for us.

If you've never seen this syntax for auth in a url like this I'm sure this looks
weird but here's how you get your URL for the webhook to add to Postmark

`https://actionmailbox:PASSWORD@example.com/rails/action_mailbox/postmark/inbound_emails`

In this example from the Rails Guides, `actionmailbox` is our user
(ActionMailbox handles that) and we inlcude our generated password we just added
to our credentials.

the `@example.com` is the URL for our server, as in the URL for where your app
is deployed (not postmark)

AM will authenticate the request and send to the Postmark inbound email
controller for processing.  That processes is covered in more detail in my
[ActionMailbox Guide].

We now have all the info we need to create our inbound webhook URL.

Click on the 'Default Inbound Stream' for your server and look for the section
'Set your server's inbound webhook URL'

^^^^^^^^ 

You could set up the inbound webhook URL on the 'Setup Instructions' page for
the Inbound Stream, but there's 1 small change we need to make that's not
visible on that page.

There's a helpful little note in the Rails guides about including raw email
content in the payload.

> When configuring your Postmark inbound webhook, be sure to check the box labeled "Include raw email content in JSON payload". Action Mailbox needs the raw email content to work.

If you navigate to the 'Settings' section for your Inbound Stream, you'll see a
checkbox in the Webhook section that says 'Include raw email content in JSON
payload'

That's what the Rails guides were referring to so make sure that option is
checked.

With that option checked, add your webhook url to the field.

Now we're added new configs, it would be a good time to check it.  You should
see a button to check the webhook URL confirming Postmark can correctly send
requests to your sever.  Clicking on the button should show a loading state and
if everything goes well, you'll see:

'Remote server returned an HTTP status code of 204'

If you're not seeing that, you know that there's an issuse connecting your Rails
app and Postmark so gives you a good idea of where to look too (check your rails
logs to see if the request is hitting your server and if so, if there's anything
obvious happening)



#### Testing and debugging locally


If you'd like to connect a domain you have to handle all your inbound mailers
(hot tip: subdomains save a ton of headace) you can read more about that here

https://postmarkapp.com/developer/user-guide/inbound/inbound-domain-forwarding

The short version is you need to create a MX record for your inbound email
domain and set the value to `inbound.postmarkapp.com`

If I wanted to set up something like `inbound.example.com` as my inbound email
URL, I would create a new DNS record, set the type to MX, add `inbound` in the
Name section, set the priority 10 (from their docs) and the value to
`inbound.postmark.com`

[SCREENSHOT OF SUBDOMAIN EXMAMPLE]

The screenshot illustrates adding the DNS record for `example.yourdomain.com`

After adding your DNS record, head back to the Inbound Stream settings page and
add your inbound domain to the 'Inbound domain forwarding' section and click
'Save Changes'

And Voila, now you have a production setup for your inbound email.

It's pretty likely that at some point, you'll have some issues you'd like to try
to diagnose or re-create in your local environment.  

Using ngrok or cloudflare secure tunnel servive exposes your local development
environment (localhost:3000) to a public URL.  This is great for testing
webhooks from a 3rd party service to your local environment.

(play around with the ngrok stuff
(See if you can get this running with ngrok (update development.rb ingress)

start ngrok with `ngrok http 3000` or `ngrok http --log=stdout 3000`

You'll also need to update your `config/environments/development.rb` file to set
the action mailbox ingress to use Postmark


```
# config/environments/development.rb
  # use ngrok and update postmark settings to test locally
  config.action_mailbox.ingress = :postmark
```

be sure to re-start your server for the changes to take effect (not positive on
that but probably)

Grab your ngrok URL, vist in your browser to make sure your app loads.

Swap your domain name for the ngrok domain in the Webhook settings, it should
look something like this:

`https://actionmailbox:7XidyHhGWk7Lir9@dbdca4d80499.ngrok.app/rails/action_mailbox/postmark/inbound_emails`

where `dbdca4d80499.ngrok.app` is the URL sending traffic to my local
environment.

If we hit the 'Check' button, even without saving, that should check the URL to
make sure it's working.  You should see a request from Postmark hit your local
environment a couple seconds after clicking the button.  If all goes well,
you'll see the green text saying Postmark received a 204 status code.

If it failed, hopefully it made it to your app and you'll have more information
in your local logs.


(Trying to send a test email to postmark email
6c122bd4809060bdccf96b593a0720ad@inbound.postmarkapp.com - waiting to see if it
hits my app. I can see the inbound emails but they're still sitting in the
queued status)

^^^^^ Derp, I don't think I saved the changes so that's probably why it was
queued for so long?  Updated and still seeing hte emails queued like 25 mins
later.

This isn't a super speedy process so you may want to just fire the email to
postmark, download the source and use in the Rails conductor

It's been like a day and emails still showing as pending in Postmark.  Maybe
just cut this and talk about getting the email source and using it in the Rails
Conductor.

Mention something about reviewing the emails in the Activity section,
downloading the source and using in the Rails Conductor.








