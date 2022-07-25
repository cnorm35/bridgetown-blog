---
layout: post
title:  Building Easter Eggs Into Phone Systems with Ruby
date:   2022-07-20 16:54:46 -0500
category: ruby
excerpt: "Creating easter egg features in a Twilio voice
phone line with a few blocks of code"

author: cody
---

What started as just a note for something to look into when I was between tasks 
and has turned into something that's been a blast to work with.

I don't know why I've been so fascinated with easter eggs has been lately.

Maybe because an easter egg is something like a secret way to identify members 
of your tribe. Something that easily gets passed by or ignored by most people, 
but really strikes a note for people that get it.  Kind of like a digital inside joke, but
one that doesn't exclude or leave anyone out.

For me, I think it's also a good representation of Ruby

It's a little weird, quirky, focuses on developer happiness, and can easily be 
useful or just for fun.

This whole experience has been a lot of fun for me.  You may see me repeating
that phrase _fun for me_ and might not think much of it, but for me, it's been a
reinvention.

I don't think I realized how much I wasn't enjoying development for the past 
couple of years.  I wasn't motivated to do much more than just get by.  It
really felt like treading water.  There's also _nothing wrong with just getting
by and doing your work_.  In the tech bubble, it can seem like everyone
everywhere is working non-stop.  But nobody can judge how you're feeling better
than you so don't feel like you're somehow obligated to code in all your free time or
publish blog posts constantly.  Go at your own pace and do what feels right.

The past year or so, I've had some ideas for things I wanted to work on that are
some [Hell Yeah](https://sive.rs/n) ideas. After blocking out some dedicated time 
each day for work only on side projects, I was able to get to my ever elusive *MIP*

That's my _Minimum Interesting Product_. The point where your
project gets over the hump of boilerplate, setup and adding some basic styles
and gets to a point where you can focus on specific features. 

That's something I started using to describe the inflection point on a project
where I keep coming back to it because all the boring stuff is done. I don't believe 
that's something like a recognized term so you might get some weird looks
if you throw that out around other people.  If they ask for more information,
tell them you made it up and begin your journey as a thought leader. I'll back you up.

SaaS templates and frameworks like [Bullet Train](https://bullettrain.co/){:target => "_blank" } 
and [Jumpstart](https://jumpstartrails.com/){:target=> "_blank"}
have made that process a lot easier and let you get to the good stuff.  I've
used both in the past and allowed me to get right to the fun stuff.

Anyway, having some non-trivial projects for the first time in a long time really
sharpened my skills and more importantly, got my wheels turning again in a way
they haven't in some time.

One of the things I wanted to add to this project was working in
an easter egg somewhere.  I wasn't sure where, or what it would be, I just had a
note about how I thought it would be fun to work in if something ever came up.

Well, it came up.

I was working with a Twilio programmable phone number sending and receiving SMS
messages and alerts. I've worked with SMS a ton in the past, but I don't think 
I've ever called one of the numbers.  So I decided to call and see what would
happen.

I got a voice response from Twilo about how I needed to configure a way to
handle the calls.  After a little research, I saw it was really similar
to how SMS is handled.  When Twilio gets an inbound SMS or Phone call, a request
is sent to your defined webhooks URL.

The main difference from the SMS webhooks is you need to send a TwiML
response to the webhook request. TwiML is basically Twilio flavored 
XML to control voice interactions.

I found a few examples of the capabilities and thought I could get a response
working without too much hassle.  I added something pretty close to their docs
to respond with a simple message saying this number was for inbound messages
only and email support at blah blah blah dot com

Instead of constructing the XML on my own, the twilio-ruby gem
has support for creating the reqiured TwiML responses.

Outside of creating a new endpoint for the voice webhook, I was able to get a
response working with a few lines of code

If you want to test this out locally, you'll need something like ngrok to point
the webooks to your local machine. It can be a bit tedious keeping those updated
so I created a post on creating a rake take to automatically set your twilio
webhook routes for voice and SMS with a rake task [here](/ruby/automating_twilio_webhook_urls){:target="_blank"}

```ruby
def parse
  response = Twilio::TwiML::VoiceResponse.new do |r|
    r.say(message: 'I wish I had a cool robot voice')
  end

  render xml: response.to_xml
end
```

I've mentioned before how much I love using established technology like SMS and
email. With just a few lines, I was able to add voice to the mix.

I started looking through some of the examples and docs and found some really
cool capabilities and options.

One of the options was different voices with Amazon Polly in addition to the
Twilio voices.  I remember thinking something along the lines of "I wish there
was a cool 80s robot-sounding voice like the computer from War Games"  [What's War Games?](https://en.wikipedia.org/wiki/WarGames)

Alas, there was not.

However... I did remember seeing an example of returning the URL of an MP3 file
that would play on the call.

That got me thinking.... maybe I could fake my own Joshua computer voice with
some sound clips. Of course, "Would you like to play a game" is what I always
think about when I hear War Games.

That phrase got me thinking about the move and playing games with the computer.
After all, he does connect with the super-computer by dialing a phone line.

All of a sudden, I had an idea for my easter egg. With some searching, I found
the phone number his modem dialed into to connect to the computer and thought 
that would be the perfect thing to start the sequence.

TwiML has the option to gather responses by either pressing buttons on
touch-tone phone (do people still say that, am I dating myself?) or speech.

For the response, I created a `gather` block around my existing phone message.
That allowed me to collect user input, without making it apparent to the caller

The gather block needs a route to redirect to.  That takes input from your phone 
call along with some other information and sends a webhook request with the 
corresponding data to that route.

The params for that request include `Digits`.  That's all I needed to check the 
input against a condition.  In this case, I was looking for the phone number 
to the computer from the movie. `3992364`


```ruby
  def parse
    response = Twilio::TwiML::VoiceResponse.new
    response.gather(action: webhooks_parse_voice_input_path) do |g|
      g.say(message: "This number is for outbound messages only.  Please email something@blah blah blah.com for assistance")
    end
    # webhooks_parse_voice_input_path is another endpoint that needs to be created
    response.redirect(webhooks_parse_voice_input_path)
    render xml: response.to_xml
  end
```

For this block, the breakdown is:
* Create a new response
* Start gathering input from the user
* Read the message saying this number is for outbound messages only and to email
support with issues
* Set the route to redirect to _after_ this request finishes
* Render XML response

In the example above, it's always going to send a request to the redirect
webhook endpoint.  My thinking was that anyone not trying to poke around would have
hung up by now.  If you would like more control over the call flow, you can
hang up the call with `response.hangup`

In the next endpoint, we start checking the inputs from the user.  There are
options for both speech and digit input but for now, it's just focusing on the
digit input.

After entering the secret code, things start to get interesting.

```ruby
  def parse_voice_input
    if params["Digits"] && params["Digits"] == "3992364"
      response = Twilio::TwiML::VoiceResponse.new do |r|
        r.gather(input: "speech", action: webhooks_parse_easter_egg_response_path) do |g|
          g.play(url: "https://s3.us-east-2.amazonaws.com/Greetings.mp3")
          g.play(url: "https://s3.us-east-2.amazonaws.com/Its+been+long+time.mp3")
          g.play(url: "https://s3.us-east-2.amazonaws.com/Play+a+game.mp3")
        end
      end
      response.redirect(webhooks_parse_easter_egg_response_path)
    end
    render xml: response.to_xml
  end
```

I mentioned before I thought about hacking together my own Joshua voice (80s
computer voice) by stringing together a few voice clips.  3 to be exact.  I
found some clips that would work and uploaded them to an S3 bucket.  The quality is
not great, but I'm thankful someone took the time to put those together and
make them free to download.

Together, the prompts are something like this:

"Greetings professor Falken"

"It's been a long time, can you explain the removal of your user account..."

"Shall we play a game?"

These clips are also wrapped in a gather block.  So we're accepting input and
will redirect to another endpoint as a handler.

After that starts, there are 3 possible outcomes. 

* Input we're checking for
* Input we're not checking for
* Caller doing nothing

Let's start with the do nothing option (the `else` condition)

If you don't do anything after hearing the response, you hit the fallback 
option that returns a cheeky response:

'Strange game, the only winning move is not to play'

This was the climax of the movie after he teaches it to play tic-tac-toe against 
itself and realizes there are no winning moves in thermonuculear war.

```ruby
  def parse_easter_egg_response
    response = if params["Digits"] == "567482"
      Twilio::TwiML::VoiceResponse.new do |r|
        r.say(message: "You have chosen wisely, email global thermonuclear war@blah blah blah.com to claim your prize. Goodbye")
      end
    elsif params["Digits"] == "325536"
      Twilio::TwiML::VoiceResponse.new do |r|
        r.say(message: "You have chosen wisely, email global thermonuclear war@blah blah blah.com to claim your prize. Goodbye")
      end
    else
      Twilio::TwiML::VoiceResponse.new do |r|
        r.play(url: "https://s3.us-east-2.amazonaws.com/strange-game-the-only-winning-move-is-not-to-play.mp3")
      end
    end
    render xml: response.to_xml
 end

```
The other two options rely on user input.

The two successful responses are entering numberic representation of 'Joshua' (567482) and
'Falken' (325536).  Professor Falken was the creator of the program and Joshua was the
name of his son who died and the secret logon to gain access.

With the code block above, there's no difference in the response, I just wanted
to have a couple of different options for success.

Entering either of the secret codes correctly plays a message with an email
address to email to claim a gift.

That's pretty much it. This is a goofy, and hopefully fun, example showing some 
power concepts. 

* Creating speech from text as a response to a caller
* Playing Mp3 files
* Handling and parsing user input and speech"

Pretty neat, right?  I thought so. After starting on the first version of this 
post, I had another idea for a goofy and fun thing to build that I couldn't seem 
to shake so decided to take some time and build it.

I'll have another post that goes into that idea in more detail soon.

