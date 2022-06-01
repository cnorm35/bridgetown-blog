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
service.

^^ Talk about how Rails 7 uses a Procfile and services by default

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

Before I start breaking everything down, here's the code for the rake task. The
dependencies for the rake task are a running [ngrok](https://ngrok.com/) instance, the [twilio-ruby](https://github.com/twilio/twilio-ruby) gem (I'm using v5.67.1), and if you'd like a pop of color in your terminal to make things easier to spot, the [colorize](https://github.com/fazibear/colorize) gem

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
        twilio_client = Twilio::Client.new
        twilio_client.api.v2010.accounts(account_sid).incoming_phone_numbers(INCOMING_PHONE_SID)
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

Now lets dig a little deeper into what's going on.  First off, there are
probably a lot of different ways to do this, but for me, it made the most sense
as a rake task.  That gives me the control to run it easily and whenever I need
it instead of something that fires off automatically whenever ngrok starts up.

Inside the rake task, the first thing we need to do is find the URL of the
running ngrok instance.  Ngrok has an api endpoint running at port 4040 for the
current tunnels.  This endpoint returns a XML response with information about
the current tunnels.  There are 2 tunnels returned, and I grab the one with the
name `command_line (http)` which is the second one returned.

Usually my ruby HTTP lib of choice is HTTParty but I wanted to set this up using
the ruby Net::HTTP lib to keep from introducing a dependency when not needed

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

If ngrok is not running, we'll get an `Errno::ECONNREFUSED` error so we can
rescue that error, and print some output in yellow to make it easy to know when
something went wrong

```ruby
  begin
    response = JSON.parse(Net::HTTP.get(URI("http://localhost:4040/api/tunnels")))
    ngrok_url = response.dig("tunnels", 1, "public_url")
  rescue Errno::ECONNREFUSED
    puts "*******NGROK not running*******".yellow
  end
```

Now we have a repeatable and reliable way to grab our ngrok url whenever it
running.  The next part is to programatically update the webhook endpoints.

It took quite a big of digging to find the endpoints needed in the twilio-ruby
gem, but once I located them, everything was pretty easy.

There's a few things we'll need to update the webhook url with twilio.  To
create the twilio client, you'll need to get your `account_sid` and
`auth_token`.  You can find both of those by logging into twilio.  While you're
there, be sure to find the service id of your phone number.  

Since this was something I added to a rails app, I add my keys to twilio in an
initializer so I can just use Twilio::Client.new anywhere in the app to create a
new api client.
