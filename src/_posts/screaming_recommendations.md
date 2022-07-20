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
interactions.  Instead of construction the XML on my own, the twilio-ruby gem
has support for creating the reqiured Twiml respones.


