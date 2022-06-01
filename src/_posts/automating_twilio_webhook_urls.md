---
layout: post
title: Automating Twilio Webhook URLs
date:   2022-05-31 16:54:46 -0500
category: ruby
excerpt: "Automate updating Twilio webhook callback urls to your running ngrok
url"
author: cody
---
Recently, I've been doing some work with Twilio.  I've always thought SMS as a
really powerful tool to app to any app's arsenal since it leverages something
users already use multiple times per day.  In a recent report, Twilio even
reported some stupid high conversion rates (post link and figures here)

When buying a phone number with Twilio.  You have callbacks for Voice and SMS.
When someone calls in, the voice callback webhook url will receive a request
from Twilio.  When someone texts the number, the messaging callback webhook url
will receive a request from Twilio.  Both channels have a fallback URL if the
first request to the webhook fails

When developing your Twilio feautres, you'll probably want to fire up your
trusty ngrok instance, grab your url, and update your webhook URLs to point to
your running ngrok instance.  If you have no idea what ngrok is, you can get
more information LINK here. Basically, it exposes your local development
environment to the outside world.

One option for updating your webhook urls is going into the twilio console,
grabbing your ngrok url, and update the webhooks url.

Since I use ngrok for a lot of things, I'ved added it to my Procfile as it's own
service

```
# Procfile.dev
web: bin/rails server -p $PORT
css: yarn build:css --watch
js: yarn build --reload
worker: bundle exec sidekiq
stripe: stripe listen --forward-to localhost:5000/webhooks/stripe
ngrok: ngrok http --log=stdout 5000
```

My only issue with this set up is whenever I restart my services, the ngrok URL
changes and needs to be updated again.  That can make for a lot of updates when
starting and stopping your services

After some digging in the Twilio documentation, I found some options to set the
webhooks urls through the ruby gem

A little more digging helped me figure out there's and endpoint you can query
while ngrok is running to get it's public url

Putting those together in a rake task allows me to grab the current public_url
of my ngrok instance and update to the running public_urls through the API via the ruby gem 

Here is an example of the rake task
```ruby
namespace :development do
  desc "updates twilio webhook urls to ngrok url"
  task set_twilio_webhooks: :environment do
    return unless Rails.env.development?
    # needs ngrok running
    begin
      response = JSON.parse(Net::HTTP.get(URI("http://localhost:4040/api/tunnels")))
      ngrok_url = response.dig("tunnels", 1, "public_url")
    rescue Errno::ECONNREFUSED
      puts "*******NGROK not running*******".yellow
    end
    if ngrok_url
      puts ngrok_url.green
      # UPDATE TO TWILIO CODE
      if TwilioService.update_webhook_urls(ngrok_url)
        puts "Webhook URLs updated!".green
      end
    end
  end
end
```
