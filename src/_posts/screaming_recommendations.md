---
layout: post
title:  Building Screaming Recommendations
date:   2022-07-20 16:54:46 -0500
category: ruby
excerpt: "What started out as a blog post about a proof of concept I made
snowballed into a fun and weird side project.  Introducing Screaming
Recommendations - call in to get recs on what to watch"
author: cody
---

This whole experience has been a lot of fun for me.  You may see me repeating
that phrase _fun for me_ and might not think much of it, but for me it's been a
revelations/re-invention.  I don't think I realized how much I wasn't enjoying
development for the past couple of years.  After blocking out some dedicated
time each day for work only on side projects, I was able to get to my ever
elusive *MIP*

That's my _Minimum Interesting Product_. The point where your
project gets over the hump of boilerplate, setup and adding some basic styles
and gets to a point where you can really focus on specific features.

SaaS templates and frameworks like Bullet Train [LINK] and Jumpstart PRO [Link]
have made that process a lot easier and let you get to the good stuff.  I've
used both (not on this project) in the past and was a great experience and let
me get right to the fun stuff.

Anyway, having some non-trival projects for the first time in a long time really
sharpened my skills and more importantly, got my wheels turning again in a way
they haven't in some time.

One of the things I wanted to add in this non-trival side project was adding in
an easter egg somewhere.  I wasn't sure where, or what it would be, just had a
note about how I thought it would be fun to work in if something ever came up.

Well, it came up.  Or at least I brought it up and snowballed from there.

I was working with a Twilio programmable phone number sending and receiving text
alerts.  I've worked with SMS a ton in the past, but I don't think I've ever
called one of the numbers.  So I did.

I got a voice response from Twilo about how I needed to configure a way to
handle the calls.  After a quick skim of the docs, I saw it was really similar
to how SMS is handled.  Twilio gets an inbound SMS or Phone call, then sends a
request to your defined webhooks url.  

The main difference from the SMS webhooks is you need to send a Twiml(format
this right) response
to the webhook request. Twiml is basically Twilio flavored sms to control voice
interactions.  

I found a few examples of the capabilities and thought I could get a response
working without too much hassle.  I added something pretty close to their docs
to respond with a simple message saying this number was for inbound messages
only and email support at blah blah blah dot com

Instead of construction the XML on my own, the twilio-ruby gem
has support for creating the reqiured Twiml respones.

Outside of creating a new endpoint for the voice webhook, I was able to get a
response working with a few lines of code

```ruby
# put response here
```

I started looking through some of their examples and docs and found some really
cool capabilites and options.  

One of the options was different voices with Amazon Polly in addition to the
Twilio voices.  I remember thinking something along the lines of "I wish there
was a cool 80s robot sounding voice like the computer from War Games"  [WIKI](https://en.wikipedia.org/wiki/WarGames)

Alas, there was not.

However, I did remember seeing an example of returning the url of an MP3 file
that would play on the call.

That got me thinking.... maybe I could fake my own Joshua computer voice with
some sound clips. Of course, "Would you like to play a game" is what I always
think about when I hear War Games.

That phrase got me thinking about the move and playing games with the computer.
After all, he does connect with the super computer by dialing a phone line.

All of a sudden, I had an idea for my easter egg.  Ater some googling, I found
the phone number his modem dialed into and thought that would be the perfect
thing to start the sequence.

Twiml has the option to gather responses through either pressing buttons on
touch-tone phone (do people still say that, am I dating myself?) or speech.

For the response, I greated a `gather` block around my existing phone message.
That made it so there way to know there was input, but it was there.  The gather
block needs a route to redirect to.  That takes input from you phone call and
sends a web hook request with the correspoding data to that route.  

The params for that request include `digits_entered` (or something like that)
so there was an easy way to check the input against some condition.  In this
case, I was looking for the phone number to the Wopr computer. NUMBER GOES HERE

```ruby
# code example
```

After entering the secret code, that's where things start to get interesting.

I mentioned before I thought about hacking together my own Joshua voice (80s
computer voice) by stringing together a few voice clips.  3 to be exact.  I
found some clips that would work and uploaded to an S3 bucket.  The quality is
not great, but an still thankful someone took the time to put those together and
make them free to download.

All together, the prompts are something like this


"Greetings professor falken"
"It's been a long time, your user account was inactive"
"Would you like to play a game?"

These clips are also wrapped in a gather block.  So we're accepting input and
will redirect to another endpoint as a handler.

After that starts, there are 3 possible outcomes. Let's start with the do nothing option. 
If you don't do anything after hearing the Joshua response, you hit the fallback option returning a cheeky response:

'Strange game, the only winning move is not to play'

This was the climax of the movie after he teaches it to play tic-tac-toe and realizes there are no winning moves in thermonuculear war.

***** update code to reflect this.

The other two options rely on user input.

If you enter the numeric representation of 'Joshua' you get one thing. If you enter the numeric representation of 'Falken' you get another response.

these options return a voice prompt saying email: globalthermonuclearwar@codynorman.com to claim your prize.

Just a heads up, there's no email address and no prize. Then again, that's exactly what someone who would set up an easter egg like that would say. Who knows?

That's pretty much it. The main concepts of "Creatings speech from text", "Playing Mp3 files", "handling and parsing user speech and input"

Pretty neat right?  I thought so.  After starting on the first version of this
post, I couldn't shake another goofy idea I had and decided I had to build it.
I thought about re-creating Kramer's inept and short-lived movie phone from
Seinfield.

CLIP HERE IF POSSIBLE

For some reason, that cracked me up and thought it would be a cool proof of
concept with relativly little code.

I started on that and started a new blog post to go with it.  Sometime in the
first day or so of working on it, I had the thought that Halloween is a few
months away and maybe could do horror movies in Oct.  For a little bit of
background, I love movies and have been hoarding movies ever since college.  I
especially love horror movies so thought that would be really fun.

I made a note of it and carried on for a bit and at some point, said to myself
"Screaming Recommendations" instead of "Sreaming Recommendations"  Decided to
check to see if the domain was available and was really excited to see it was
available.  It doesn't take much for me to buy a domain name and this was no
exception.

After getting the domain, I fired up a fresh rails app and was on my way.
I wanted to create a phone line that you could call in, get a couple prompts for
different services, make a choice and get some recommendations on the most
popular horror movies for streaming.  I wanted something weird, fun, and
moderately useful (debatable).  Think a cross between a late night public access
horror show and a moviephone.  

I remember getting recommendations on great stuff from people working at the
video store and thought this was a cool throwback to that.  It's not there yet,
a not doing any AI or machine learning, just returning popular results but think
it would be easy to improve the recommendaitons with some more work.


The voice was fine, but was not putting out what I would call a spooky vibe so
thought about some possible solutions.  Turns out, Fiverr has some great options
for getting voice over work done so found someone to do an intro i could play at
the start of the call.
