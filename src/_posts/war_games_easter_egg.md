---
layout: post
title: Automating Updates to Twilio Webhook URLs
date:   2022-07-11 16:54:46 -0500
category: ruby
excerpt: "Wasting time with easter eggs"
author: cody
---

This was a really fun one.  Thinking about writing about this post was one of
the main factors of getting a blog back up just so I could talk about this.  

Basically, it went something like this...

I was going to add a simple voice response with Twiml saying something along the
lines of "This number is for outbound messaging only, please email blah blah
blah for assistance.  Simple enough, right?

When looking over the docs for the response, I started looking through the list
and samples of the different voice options offered through AWS.  For some
reason, my frist thought was something along the lines of "I really hope/wished
there was a robot voice like the computer in war games"

Alas, there was not.  No biggie, and moving on.

A couple pages later in the docs, I saw an example returning the url of an audio
file in the Twiml response and their song of choice was rick astley's Never
gonna give you up.

That got my wheels turning.  After seeing some more examples on collecting input
from inputting numbers on your phone, I had an idea to make my own Joshua voice.

Some quick searching for sound bites matching some phrases I was looking for and
I found what I was looking for.

I threw the sound bites in an S3 bucket I had and was able to get it to play
the responses I was lookings for.  The quality isn't great, but it get's the job
done and I'm thankful someone took the time to make what I was looking for.

That gave me all the ingredients to embark down this quirky rabbit hole.  It was
somethng weird and whimsical and that was really relevant to ruby.

First, a bit of back story.  If you're not familar with War Games, it's one of
my long time favs and always get a kick looking back at the technology of the
time.  I'm also a sucker for that '80's conputer/robot voice Joshua uses' (also,
see roberberbles from thundercats)

In the movie, he starts 'War Dialing' trying to find open connections and open
networks he can snoopp around with. One of the numbers he dials into is for a
supercoputer at NORAD and almosts starts world war 3.

Anyway, lets focus more on how the easter egg is set up


Basically, you call the Twilio phone number, it plays a voice prompt saying it's
for outbound numbers only and to email instead.  Pretty cool, but not very
exciting.

After the prompt, if you enter '3992364' according to IMDB trivia, that was the
phone number
https://m.imdb.com/title/tt0086567/trivia/?ref_=tt_ql_trv

> The phone number that David used to call the NORAD WOPR computer was 399-2364.

If you enter that number, you get some prompts of strung together sound clips
mimicking the WOPR computer.  Something along the lines of 

"Greetings professor falken"
"It's been a long time, your user account was inactive"
"Would you like to play a game?"


After that starts, there are 3 possible outcomes. Let's start with the do
nothing option.  If you don't do anything after hearing the Joshua response, you
hit the fallback option returning a cheeky response:

'Strange game, the only winning move is not to play'

This was the climax of the movie after he teaches it to play tic-tac-toe and
realizes there are no winning moves in thermonuculear war.

***** update code to reflect this.

The other two options rely on user input.

If you enter the numeric representation of 'Joshua' you get one thing.  If you
enter the numeric representation of 'Falken' you get another response.

these options return a voice prompt saying email:
globalthermonuclearwar@codynorman.com to claim your prize.  

Just a heads up, there's no email address and no prize.  Then again, that's
_exactly_ what someone who would set up an easter egg like that would say.  Who
knows?

That's pretty much it.  The main concepts of "Creatings speech from text",
"Playing Mp3 files", "handling and parsing user speech and input"

Are all some really power features.  That lets you do things like route phone
calls, play custom messages while customers are on hold or any other VOIP
feature you might need.

About a week after creating this easter egg, I was watching some sienfield at
the dentist office and got an idea for another one and would really like to see
someone build it or I'll try to throw something together for it.

Kramer's movie phone line.

I don't want to get sued, so maybe we'lll build a new-age version of the old
movie phone.
