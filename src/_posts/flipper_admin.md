---
layout: post
title:  Moving fast and breaking things, then fixing them.
date:   2023-0307 16:54:46 -0500
category: ruby
excerpt: "Using feature flags to protect your code from yourself"
author: cody
---

I have a couple of different side projects going at any given time.  Each one of
those projects usually has a couple of different environments (staging and
prod) with different states of data.

All of these projects are ran by yours truly.  Being solo, I have to try to get
as much done as possible with less.  Time is always at a premium for me.

Most of the time, when I'm working on a sideproject, I try to timebox things.  I
feel like going into my work with a set time limit, it forces me to try to get
as much done as I can in the time perioid and cultivate a ruthless focus.

Sometimes, my monkey brain needs that little hit of Dopamine by that sweet
feeling from movong one of my Trello cards into the 'Done' lane.  For that to
happen, I need to deploy my changes.

Fuck it ship it right?

Well, the Fuck it part seems to hold true.  With different environments in
differnt states, sometimes I can break things.  Even with CI, and pretty decent
test coverage, sometimes some things happen.

### Feature Flags
Enter Fetature Flags.  If you're not familiar with the concept of a feature flag
the basic idea is to have some 'Feature' in either an active or inactive state.  This state can be used for control flow in your code base.

My Feature Flag of choice has been Flipper LINK

The basic syntax for a feature flag with Flipper is somehing like

```ruby
if Flipper.enabled? :thing
  # do the thing
end
```

Then in your console, initializer or somewhere similar, you can toggle the
feature with

```ruby
Flipper.enable "thing
=>
```

You can read more about Flipper and feature flags here LINK


Feature Flags are a great way to test, validate and slowly roll out features, no
argument here.

The missing element for me was Flipper UI mounted within my Admin app.

After wrapping an experimental feature in a Feature flag, I noticed something
was off and after looking into the issue, I had mispelled the name of the
feature when I was typing it in manually in a Rails console.

After that little snafu, I tbought there had to be a better way to handle it and
didn't take long to RTFM and find Flipper UI.

Think of Flipper UI like and admin engine for your feature flags.  Not only does
it give you the options to create feautres in a simple web ui, but you can also
get some really granular control on who and how it's rolled out.


SCREENSHOTS HERE

Things like rolling out to 30% of the users %10 of the time are available with
just a few clicks.

Now, within the admin, I have a section to create new features, toggle on and
off and specify how I would like things rolled out.

But if I'm being honest, most of the time, I broke something and making the
feature inactve with a couple of clicks fixes the issue without having to revert
changes and wait for them to be deployed.

For me, it's a little additional peice of mind that I can ship quickly while
still having an easy way to revert changes.  You should look into it it's pretty
cool.
