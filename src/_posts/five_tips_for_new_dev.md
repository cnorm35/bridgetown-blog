---
layout: post
title:  5 tips for new Rails develoopers
date:   2023-06-02 16:54:46 -0500
category: ruby
excerpt: "5 tips for new Rails developers to level up your game"
author: cody
---

I’ve been fortunate enough to have worked along side a top notch group of newer developers the past few months.  It’s been the first time in a few years that I’ve regularly worked with new developers and has been a great chance to re-examine and question how I do a lot of things.

We conduct morning stand ups where everyone has a chance to discuss their issues they’re working on and any blockers they may have.  We run through a normal standup and then take the remaining time we have (usually from my free zoom account, please don’t hate) we’ll try to review a recent issue or fix in more detail or see if the group has any questions and would like to go into more detail on something.  These talks range from How Concerns work and how ot create and use them?  Where is the best place for this test and what’s needed to set it up? And even the occasional live coding session where I typically drive and talk about what I’m doing.  Rebasing a branch and cleaning up some commits is a typical example of that one.

It seems like every day I have a “Oh by the way…” that leads into some examples or tips I’ve learned, usually painfully, over the years.

This is by no means exhaustive, but here are a few tips I’ve found myself talking about a lot recently.

You should probably be using your Rails console more.  
This one might be a little opinionated. I’m a hands on learner and _always_ am poking around in the Rails console.  Getting a running Rails console and pulling a model is usually the first thing I do after getting a new app up I’m onboarding to and running.  I have a completely unsubstantiated theory that with the advent of VSCode and more editors with a IDE feel with everything built in has made it seem like the Rails console is an afterthought and not a core feature.  *Shakes fist* back in my day (not sure about that one)

You can also customize your rails console by updating your .irbrc file.  Some of the things you can customize are automatically expanding objects or disable the autocomplete available in the newer versions of Ruby.

Some examples of common things I do with my console are:
Testing the output of helper methods by using `helper`

helper.number_to_currencty(123)

Seeing the output from the I18n helper is another one, especially where then are inputs involved.

Testing out queries

A quick smoke test making sure a new class or module has been added.  An example of this would be creating a concern called MySpecialConcern and calling that in the console.  If it’s defined correctly, it will return itself.

Source location to find where things are defined

Practice using more debugging tools.  
This one is admittedly a little more tricky than it used to be if you run your Rails application with bin/dev using something like foreman.

In this scenario, you’ll place your debugger (pry, debugger, byebug or whatever) and your server process will pause and display a debugger prompt that you’re unable to connect to.  Personally, I’m a big fan of overmind from the fine folks at Evil Martians which will let me connect to a specific process in my Procfile.  What does all that mean?  Let say I start my Rails server in a process I’m calling ‘web’ , I have some other services that handle things like rebuilding js and css, and sometimes extras like stripe cli, docker, or ngrok.  

I can connect to any one of these individual processes with:

Overmind connect web.

This connects me to my web process with overmind.  This is along the same lines of if I just started my Rails server with rails s.

That connects me to the debugger.

Overmind requires tmux, which dont get my wrong is one of my favorite tools, but it’s still an additional dependency and one more thing to install and mange as a newer dev.

I think the easiest way to get to a live debugger is something like commenting out the web process in your Procfile, open a new window for your terminal and start a Rails server.  In this window your debugger will be available.

Once you _have_ the debugger repl running?

Usually I check the current state of variables and objects, sometimes I’ll build new ones and check valid? And errors before trying to save and raising errors.  That lets me try to set some values and see if I know what’s required to get our object to a valid state.

There are a ton of options you can use.  I don’t use much more past ‘c’ for continue meaning continue execution, and ‘n’ for next when goes to the next line.

The biggest benefit for me is using it as a quick way to re-create the exact conditions of an error making for a much shorter feedback loop

Communication
This is a pretty broad one.  Can be anything from writing a good commit message, a good pull request description, comment for clarification on an issue, how to clarify feedback and approach and even how to talk about what you learned in more detail to share with others


Git gud
This isn’t even about using Git.  It’s more about figuring out how to find more information on whatever it is that your looking for wherever the repo is hosted.  For me, that’s usually github.  One thing I suggest is before you open your first pull request, take some time to look through a lot of the recently merged changes to get a better understanding of how the team prefers to do things.  Branch naming conventions, what type and how much testing do they require, screenshots or recordings that may be needed and any other specific info that might not be 100% in a pull request template, if it’s even there at all.

Being able to search through source code for gems your using to see how things are defined, or seeings how they’re implemented and tested

Searching the git blame for history on changes

Searching by a specific term or feature and filtering by closed pull requests to see what’s been merged.


Don’t fly too close to the sun and try to add a giant block in a spec at once
This is a continuation of keeping your feedback loop quick.  I can’t tell you how many times I’ve tried to implement a whole spec with contexts, describe, before blocks of set up only to leave out an end or two.

A good habit I always recommend is adding your blocks one at a time and running your spec to make sure everything is still working.

It may sound like overkill, but a few extra seconds and finding errors fast and after minimal changes beats commenting out and running things to try to get it all working.  Run often and fail fast.


Like I said, this is by no means exhaustive or definitive, just some things I’ve found myself talking about on a few occasions recently.  If there are some tips you’ve found really useful for becoming a more comfortable and confident rails developer, please let me know, I’d love to hear them.


***** Ask for feedback on favorite tip and write a more in-depth article for the most popular ones first.
