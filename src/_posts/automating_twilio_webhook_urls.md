---
layout: post
title: Automating Twilio Webhook URLs
date:   2022-05-31 16:54:46 -0500
category: ruby
excerpt: "Automate updating Twilio webhook callback urls to your running ngrok
url"
author: cody
---
### Sms is a great tool
Recently, I've been doing some work with Twilio.  I've always thought SMS as a
really powerful tool to app to any app's arsenal since it leverages something
users already use multiple times per day.  In a recent report, Twilio even
reported some stupid high conversion rates (post link and figures here)

Twilio phone numbers have webhooks for both Voice and SMS. These webhooks are
called whenever your Twilio phone number gets an inbound text or phone call.

When someone calls in, the voice callback webhook url will receive a request
from Twilio.  When someone texts the number, the messaging callback webhook url
will receive a request from Twilio.

Both channels have a fallback URL if the first request to the webhook fails.

### Sending Webhook Requests To Your Local Environment

If you've ever developed features from a third party webhook with your local
environemnt, you probably did it by exposing your local environment to the
internet with something like ngrok.

If you haven't heard of ngrok, it's one of my favorite tools that I've been
integrating into more workflow more and more.  You can read more about ngrok
[here](https://ngrok.com/).


### Running Ngrok In Your Procfile

With Rails 7 really embracing using foreman and Procfiles for starting and running
services, I've added ngrok as it's own process so it's always available.

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

### Updating Twilio Webhook Urls
One option for updating your webhook urls is grabbing your ngrok url, going into the Twilio console and update the webhook urls for voice and sms.

That was fine for me...for a while.

My main issue with this set up is whenever I restart my services, the ngrok URL
changes and needs to be updated again.  That can make for a lot of updates when
starting and stopping your services and errors when forgetting to update the
urls each time. Also, if that's something you're not doing very often, it's easy
to forget how to find the section to update the webhook urls.

That's when I got the itch to find a way to automate that whenever I needed it.

After some digging in the Twilio documentation, I found some options to set the
webhooks urls through their [ruby gem](https://github.com/twilio/twilio-ruby)

A little more digging helped me find an ngrok endpoint you can query to get it's public url.

Now we're cooking with gas.  With the two main parts we need, the next step is
to decide how to exeute that code when we needed it. For me, that was a rake
task.  That gives me a nice repeatable way to update the urls whenever I need
to.

### Rake Task to update

Before I start going into mroe detail, here's the code for the rake task. The
dependencies for the rake task are a running [ngrok](https://ngrok.com/) instance, the [twilio-ruby](https://github.com/twilio/twilio-ruby) gem (I'm using v5.67.1), and if you'd like a pop of color in your terminal to make things easier to spot, the [colorize](https://github.com/fazibear/colorize) gem

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

Now lets dig a little deeper into what's going on.

Inside the rake task, the first thing we need to do is find the URL of the
running ngrok instance.  Ngrok has an api endpoint running at port 4040 for the
current tunnels.  This endpoint returns a XML response with information about
the current tunnels.  There are 2 tunnels returned, and I grab the one with the
name `command_line (http)` which is the second one returned.

Usually my ruby HTTP lib of choice is HTTParty but I wanted to set this up using
the ruby Net::HTTP lib to keep from introducing another dependency. We'll make
the request to the ngrok endpoint and parse the XML response.  Here's are some
examples of the resposnes it sends back

Truncated Ngrok response
```xml
<tunnelListResource>
  <Tunnels>
  </Tunnels>
  <Tunnels>
  </Tunnels>
  <URI>/api/tunnels</URI>
</tunnelListResource>
```

Single Tunnel Object

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

The rake task takes the parsed XML and grabs the 2nd Tunnel retured.  Why you
ask?  

It was most likely the first thing that worked from StackOverflow and I moved
on.  The `<PublicURL>` for both tunnels seem to always be the same so it's
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

Now we have a repeatable and reliable way to grab our ngrok url whenever it is
running.  We also get some colored output to give us a quick visual indicator
that everything looks a-ok.

The next part is to programatically update the Twilio webhook endpoints with our
public url.

It took quite a big of digging to find the endpoints needed in the twilio-ruby
gem, but once I found them, everything was pretty smooth.

There's a few things we'll need to update the webhook urls for twilio.  The
first thing we need is a client with our keys.

To create the twilio client, you'll need to get your `account_sid` and
`auth_token`.  You can find both of those by logging into twilio.  While you're
there, be sure to find the service id of your phone number. We'll need that to
target the phone number for the webhooks.


Twilio Updates

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
something that would work as-is without additional configuration.  I typically
configure my Twilio client in an initializer so keep duplication to a minimum

example `config/initializers/twilio.rb` file

```ruby
Twilio.configure do |config|
  config.account_sid = Rails.application.credentials.dig(:twilio)[:account_sid]
  config.auth_token = Rails.application.credentials.dig(:twilio)[:auth_token]
end
```

With the client created, we can use the `ngrok_url` combined with the endpoint
that points to your webhook handler and update our Twilio webhook urls.

If everything goes as planned, we'll output a success message in green.

If things _don't_ go as planned, we're rescuing `Twilio::REST::RestError` and
outputting the error.

With everything put together, when we need to update our Twilio webhook urls (or
any other 3rd party service in the future) we can run:

```bash

$ bin/rails development:set_twilio_webhooks
```

Finding a way to automatically run ngrok in my Procfile and have a way to
programatically grab it's public url was a great find and is something that can
be expanded to tons of other services you may like to test in your development
environment.
