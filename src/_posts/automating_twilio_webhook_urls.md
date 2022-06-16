---
layout: post
title: Automating Updates to Twilio Webhook URLs
date:   2022-06-15 16:54:46 -0500
category: ruby
excerpt: "After forgetting to update Twilio webhook URLs to ngrok pointing to
my local environment one too many times, I wanted to see if there was a way to automate
that process and not have to worry about it again."
author: cody
---
This was an interesting challenge I ran into recently and has been the 
first time in a while that I've felt that _itch_ to solve a problem  just
because I wanted to see if I could do it.

It's been (_checks notes_) about _7
years_ since the last time I wrote anything related to ruby and probably 5 years
since I've given any talks or presentations.

I don't think I realized how burnt out I was after a few tumultuous years both
personally and professionally. It was great to feel like I _had_ to solve a
problem just for fun and was even compelled to write about it.

That being said, ya boy is _rusty_, so bear with me.

Recently, I've been doing some work on a side project that uses Twilio for sending and receiving text messages.
I've always thought of SMS as a really powerful tool to add to any app's arsenal since it leverages something
users already use multiple times per day.  In a recent report, Twilio even
reported that _**96% of messages are read within three minutes of receipt, and 90% within three seconds!**_

This post won't go into detail on sending text messages.  If you'd like some
more information on how to send text messages, the
[twilio-ruby](https://github.com/twilio/twilio-ruby#send-an-sms){:target="_blank"} gem has some examples.

Twilio phone numbers have webhooks for both Voice and SMS. These webhooks are
called whenever your Twilio phone number gets an inbound text or phone call.

[Incoming message example](https://www.twilio.com/docs/usage/webhooks/sms-webhooks#type-1-incoming-message){:target="_blank"}

Both channels have a fallback URL if the first request to the webhook fails.

### Sending Webhook Requests To Your Local Environment

If you've ever developed features from a third-party webhook or service with your local
environment, you probably did it by exposing your local environment to the
internet with something like ngrok.

If you haven't heard of ngrok, it's one of my favorite tools that I've been
integrating into more workflow more and more.  You can read more about ngrok
[here](https://ngrok.com/){:target="_blank"}.


### Running Ngrok In Your Procfile

With Rails 7 embracing using foreman and Procfiles for starting and running
services, I've added ngrok as its own process so it's always available.

example `Procfile.dev`
```

# Procfile.dev
web: bin/rails server -p $PORT
css: yarn build:css --watch
js: yarn build --reload
worker: bundle exec sidekiq
stripe: stripe listen --forward-to $PORT/webhooks/stripe
ngrok: ngrok http --log=stdout $PORT
```

### Updating Twilio Webhook URLs
With ngrok running, there's now a URL to point your webhooks to.
One option for updating your webhook URLs is grabbing your ngrok URL, going into
the Twilio console and update the webhook URLs for voice and sms.

That was fine for me...for a while.

My main issue with this setup is whenever I restart my services, the ngrok URL
changes and needs to be updated again.  That can make for a lot of updates when
starting and stopping your services and errors when forgetting to update the
URLs each time.

Also, if that's something you're not doing very often, it's easy
to forget how to find the section to update the webhook URLs.

That's when I felt the urge to find a way to automate that whenever I needed it.

After some digging in the Twilio documentation, I found some options to set the
webhooks URLs through their [ruby gem](https://github.com/twilio/twilio-ruby){:target="_blank"}

A little more digging helped me find an endpoint for ngrok you can query to get
it's public url.

Now we're cooking with gas.  With the URL of our local ngrok instance and a way
to programmatically update the webhook URLs, we have the 2 pieces we need.

The next step is to decide how to execute that code when we need updates.
For me, that was a rake task.  That gives me a nice repeatable way to update the
URLs whenever I need to.

### Rake Task For Updating URLs

Before I start going into more detail, here's the code for the rake task. The 
dependencies for the rake task are a running ngrok instance, the `twilio-ruby` 
gem (I'm using v5.67.1), 
and if you'd like a pop of color in your terminal to make things easier to spot, the [colorize](https://github.com/fazibear/colorize){:target="_blank"} gem

Here is an example of the rake task
```ruby
namespace :development do
  desc "updates twilio webhook urls to ngrok url"
  task set_twilio_webhooks: :environment do
    return unless Rails.env.development?
    begin
      response = JSON.parse(Net::HTTP.get(URI("http://localhost:4040/api/tunnels")))
      ngrok_url = response.dig("tunnels", 1, "public_url")
    rescue Errno::ECONNREFUSED
      puts "*******NGROK not running*******".yellow
    end
    if ngrok_url
      puts ngrok_url.green
      account_sid = Rails.application.credentials.dig(:twilio)[:account_sid]
      auth_token = Rails.application.credentials.dig(:twilio)[:auth_token]
      development_incoming_phone_number = Rails.application.credentials.dig(:twilio)[:auth_token]
      twilio_client = Twilio::Client.new(account_sid, auth_token)
      twilio_client.api.v2010.accounts(account_sid).incoming_phone_numbers(developemnt_incoming_phone_number)
        .update(sms_url: "#{ngrok_url}/webhooks/twilio",
          sms_fallback_url: "#{ngrok_url}/webhooks/twilio_fallback",
          voice_url: "#{ngrok_url}/webhooks/twilio_voice",
          voice_fallback_url: "#{ngrok_url}/webhooks/twilio_voice_fallback")
      puts "Webhook URLs updated!".green
    rescue Twilio::REST::RestError => e
      puts e
    end
    end
  end
end
```

### The Deets

Now, let's dig a little deeper into what's going on.

Inside the rake task, the first thing I'm doing is making sure we're in the
`development` environment and returning from the task if we're not.  Probably not
required, but I thought it was a nice check to keep from accidentally updating
information outside of our `development` environment.

After ensuring we're in our dev environment, we need to do is find the URL of the
running ngrok instance.  Ngrok has an API endpoint running at port `4040` for the
current tunnels.  This endpoint returns an XML response with information about
the current tunnels.  There are 2 tunnels returned, and I grab the one with the
name `command_line (http)` which is the second one returned.

Usually, my ruby HTTP library of choice is [HTTParty](https://github.com/jnunemaker/httparty){:target="_blank"} but I wanted to set this up using
the ruby Net::HTTP lib to keep from introducing another dependency.

After ensuring we're in the `development` environment, Net::HTTP then makes a 
request to the ngrok endpoint and parses the XML response.

Here are some examples of the response it sends back.

*Truncated Ngrok response*

```xml
<tunnelListResource>
  <Tunnels>
  </Tunnels>
  <Tunnels>
  </Tunnels>
  <URI>/api/tunnels</URI>
</tunnelListResource>
```

*Single Tunnel Object*

```xml
<Tunnels>
  <Name>command_line</Name>
  <URI>/api/tunnels/command_line</URI>
  <PublicURL>https://7be0-72-42-102-194.ngrok.io</PublicURL>
  <Proto>https</Proto>
  <Config>
  <Addr>http://localhost:80</Addr>
  <Inspect>true</Inspect>
  </Config>
  <Metrics>
  <Conns>
  </Conns>
  <HTTP>
  <Count>0</Count>
  <Rate1>0</Rate1>
  <Rate5>0</Rate5>
  <Rate15>0</Rate15>
  <P50>0</P50>
  <P90>0</P90>
  <P95>0</P95>
  <P99>0</P99>
  </HTTP>
  </Metrics>
</Tunnels>
```

The rake task takes the parsed XML and grabs the 2nd Tunnel returned.  Why you
ask?

Honestly, it was most likely the first thing that worked from StackOverflow and I moved
on.

The `<PublicURL>` for both tunnels seem to always be the same so it's
possible either would work.

If ngrok is not running, we'll get an `Errno::ECONNREFUSED` error so we can
rescue that error, and print some output in yellow to make it easy to know when
something went wrong

#### Grab Ngrok Public Url

```ruby
  begin
    response = JSON.parse(Net::HTTP.get(URI("http://localhost:4040/api/tunnels")))
    ngrok_url = response.dig("tunnels", 1, "public_url")
  rescue Errno::ECONNREFUSED
    puts "*******NGROK not running*******".yellow
  end
```

Now we have a repeatable and reliable way to grab our ngrok URL whenever it is
running.  We also get some colored output to give us a quick visual indicator
that everything looks a-ok.

The next part is to programmatically update the Twilio webhook endpoints with our
public URL for ngrok.

It took quite a bit of digging to find the endpoints needed in the twilio-ruby
gem, but once I found them, it was smooth sailing from there.

#### Updating with the Twilio API

There are a few things we'll need to update the webhook URLs for Twilio.  The
first thing we need is a client with our keys.

To create the Twilio client, you'll need to get your `account_sid` and
`auth_token`.  You can find both of those by logging into Twilio.  While you're
there, be sure to find the service id of your phone number. That's required to
target the correct phone number for the updates.

_Twilio Updates_

```ruby
  account_sid = Rails.application.credentials.dig(:twilio)[:account_sid]
  auth_token = Rails.application.credentials.dig(:twilio)[:auth_token]
  development_incoming_phone_number = Rails.application.credentials.dig(:twilio)[:auth_token]
  twilio_client = Twilio::Client.new(account_sid, auth_token)
  twilio_client.api.v2010.accounts(account_sid).incoming_phone_numbers(developemnt_incoming_phone_number)
    .update(sms_url: "#{ngrok_url}/webhooks/twilio",
      sms_fallback_url: "#{ngrok_url}/webhooks/twilio_fallback",
      voice_url: "#{ngrok_url}/webhooks/twilio_voice",
      voice_fallback_url: "#{ngrok_url}/webhooks/twilio_voice_fallback")
  puts "Webhook URLs updated!".green
rescue Twilio::REST::RestError => e
  puts e
end
```

I included the Twilio client setup in the rake task, but that was mostly to have
an example that would work as-is without additional configuration.  I typically
configure my Twilio client in an initializer to keep duplication to a minimum.

example `config/initializers/twilio.rb` file

```ruby
Twilio.configure do |config|
  config.account_sid = Rails.application.credentials.dig(:twilio)[:account_sid]
  config.auth_token = Rails.application.credentials.dig(:twilio)[:auth_token]
end
```

With the client created, we can use the `ngrok_url` combined with the endpoint
that points to your webhook handler and update our Twilio webhook URLs.

If everything goes as planned, we'll output a success message in green.

If things _don't_ go as planned, we're rescuing `Twilio::REST::RestError` and
outputting the error.

Now, whenever we need to update our Twilio webhook URLs (or
any other 3rd party service in the future) we can run:

```bash

$ bin/rails development:set_twilio_webhooks
```

Finding a way to automatically run ngrok in my Procfile and have a way to
programmatically grab its public URL was a great find for my workflow. This is 
something that can
be expanded to tons of other services you may like to work with in your development
environment.

Let me know any other services you automate!
