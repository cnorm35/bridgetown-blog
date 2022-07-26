---
layout: post
title: Creating a horror movie suggestion line with Rubt on Rails
date:   2022-07-25 16:54:46 -0500
category: ruby
expert: "Automated phone line, SMS and email suggestions for the most popular
horror movies available to stream"

author: cody
---

COPIED OVER FROM OLD POST

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


START of NEW UPDATES

You never know when inspiration will hit, sometimes it's right after making a
lame pun.  Regardless of where or how, when inspiration calls, answer. It seems
like I've been in some kind of funk for the past couple of years.  Created
hardly any blog posts and pretty much zero coding just for the sake of it.  even
when there was a reason, it would be hard pressed for me think of a time when I
was coding when someone wasn't paying me to.  That's fine, but not normal for
 me. The fact of the matter is, I fucking hated coding.  There I said it.  I had come
to resent the thing that I once really loved.  The thing that I packed up a
suitcase and slept on couches, air mattresses, bunk beds and floors for months
so I could learn and get a career in it.  Then at some point, I couldn't care
less.

I think that's why I'm so ecstatic that I've found a spark again.  It's wild how
much difference I feel now. Now, you may ask, what's with all the fluffy
bullshit?  We'll it's becasue I've somehow pulled myself out of that hole.

There are a lot of differences I see these days, but I think the most stark is
that I feel like the things I work on and create are 'me' again.  That's the
best compliment I can think to give myself.  I've created a fun little project
over the past couple of weeks in my free time that feel very me.  It didn't
start out that way, it started out as a simple little sidequest to see if I
could pull it off, and eventually evolved into something that I that I _had_ to
build.  

One of the most important things to riding the wave of the muse and making cool
shit is how ideas and things just seem to flow right out of you.

This seemed to flow right out of me and feels like the most 'me' thing I've
worked on for a long, long time, maybe ever.

Originally, this project started out as a small easter egg I added into a phone
system, you can read more about that HERE().  The first time I started working
on a blog post to talk more about it, I kept getting ideas about other cool
stuff to build to show off how you can parse user input through voice and sms on
phone lines. 

After reading more about the options for parsing speech for input, for some
reason, a line from Seinfield was stuck in my head.  Kramer is running his own
inepet version of moviefone out of his apartment and at one point, when trying
to guess the keys George input, gives up and says:

"Why don't you just tell me the name of the movie you selected..."

[Movie Clip](https://www.youtube.com/watch?v=qM79_itR0Nc)

I thought it would be fun to build a copy of Kramer's moviefone.  Something that
pretended to try to find a movie and eventually just gave up and asked what
movie you wanted to watch.

After getting a couple of commits in, I had flashes of manilla envenlops stuffed
with cease and desist letters from the creators and even though would be pretty
harmless, building something 100% off of someone eles's IP didn't seem right for
some reason.  I don't think there's anything wrong with that, it just felt that
it would be really artifical and short-lived without much option to expand on it
down the line.

So, I decided to build something like a re-creation of a moviefone, but for
streaming options.  If you have no idea what I'm talking about, back in the
1900s, we used to have to call movie theatres to get an automated line that
listed showtimes.  If you lived in a small town like I did, it can be next to
impossible to get through on a weekend evening.  Blowing up the phone line for
my local movie theater is something I still remember doing pretty well.  

The idea would be something like a moviefone, but with options for streaming.
If you've ever sat around with your partner trying to figure out what to watch,
you'll welcome any help you can get, especially if it's from a nuetral party
like a phone line.  

Off I went.

I got about 1 day into it before a pun de-railed my whole project. I was riffing
on some ideas and thought about how Oct will be here pretty soon and maybe I
could do options for horror movies available to stream.  At some point, I
muttered to myself something along the lines of "ha Screaming Recommendations
instead of Streaming Recommendations".  I had a nice chuckle and moved on.  For
a bit.  I decided to check if the domain was available and thought surely
someone had nabbed that one already.

Well, they hadn't.

I don't know if you're familiar with internet law 2847.32 but it states that if
you have an idea and the .com is available, you're legally obligated to by it.
I'm a stickler for the rules (not at all) so I bought it and off I went.

I decided I would build it and kept thinking of things to do with it.  It was
like the ideas were literally falling out of me as fast as I could write them
down.  Most of those ideas were crap, but the fact that they were pouring out of
me like that was a sign that I was on the right track.  I'm not the king of
overdoing things, but I'm at least an Earl, maybe a Duke of overdoing so set my
self a timelimit to get things live.  1 week for cranking out features, 1 week
for setting up and configuring production services.

Two of the first features that made the cut were adding sms and inbound email.
I was already doing voice and thought adding some inbound SMS parsing would be a
nice compliment.  For the inbound mail, I thought it would be a great chance to
show of ActionMailbox.  ActionMailbox is one of my favorite features to be added
to Rails in the past couple of years, but haven't seen much done with it so
thought this would be a cool chance to do something a little different.

So the idea was choose a handful of services, ones that I currently subscribed
to to be able to check validity of recommendations and have another channel to
verify the data I was using.

If I haven't already mentioned, I'm a huge movie lover, including horror.  So
this was right up my alley and just felt right.

Originally, I was pulling data and information from some APIs, but it looked
like the popularity metrics they were using for sorting the most popular results
are cumulative, not something that could probably be closer the 'trending'.
That was fine, but it doesn't look like that causes much change or variety in
the returned results.

In other words, if I did something like pulling the 25 most popular horror
movies for a streaming service.  The next week, there would probably be _at
least_ 22 movies that were also included the week before.  That made it seem
like recommendations and suggestions could get stale really quick.  I was
planning to eat my own dogfood and use this too so I wanted to make sure there
were better results.

I wanted to keep the app as basic as possible so didn't want to build a whole UI
for creating and updating movie recommendations so I built a CSV import and
created a spreadsheet to update the recommendation info there and import the
changes.

After having a plan for the movies, I started working working on how to create
and retrieve the recommendastions.  What I ended up with was a simple join table
called Recommendations with `movie_id` and `provider_id` as the main columns and
included the timestamps to fitler by date range.

The provider_id gives me an easy way to scope to a specific streaming provider.

I added the excellent `by_star` gem from Ryan Bigg for easy time-based filters
which now allows be to do something like:

`Recommendation.netflix.past_week` 

That returns recommmdations for Netflix creatd within the last week.  At one
point, I had a scheduled job to fetch new results and create new
recommendastions on a set schedule, but after seeing the data be pretty
consistent, seemed like it was overkill.

Added Postmark to send email replies and parse incoming mail.  It really is
amazing how easy it is to configure and deploy and inbound mail server thanks
to the work people have done on the ingresses, both offical and unoffical (I
looked at the AWS one for a bit as well, no longer official)

I also decided to deploy to render for the first time. I was a little tricky
getting started since their docs are split between deploying with or without
sidekiq since sidekiq runs as it's own worker service which I don't think is
avialable on the free tier.  I also ran into a gotcha with ActionMailbox.  It
leverages ActiveStorage for storage and retreival of mail objects.  Inbound
mail was being rounted to my rails app, picked up by the queue for processing
and routing.  However, since the background worker was a separate service, it
wasn't able to find the stored mail on disk.  i.e. it kept looking for a mail
object that was saved to the disk of the web service, not the background job
service so kept throwing errors about not being able to find the mail.
Fortunately, that was fixed pretty easily by added S3 for active storage and
just letting that handle it since i'm only going to be doing emails and they get
deleted after the inbound processing is completed.



