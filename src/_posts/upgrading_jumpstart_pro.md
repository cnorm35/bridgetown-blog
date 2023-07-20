---
layout: post
title:  Upgrading your Jumpstart Pro App
date:   2023-07-19 16:54:46 -0500
category: ruby
excerpt: "Details on how to pull upstream changes to your Jumpstart Pro SaaS
app."
author: cody
---


It's something I'm sure most people reading this post is familiar with.  You
started working on your latest idea from an app.  Start out crushing it and
shipping code left and right.  Then...as it tends to do, life gets in the way.

Blowing the dust off your old side project and getting things updated again can
be a pretty daunting task.  There's a point most of us hit where we start
wondering if the ol' _kill it with fire_ method is the best path forward and
start thinking about scrapping what you have and starting over.

I've been there so many times.  I've also been on projects that go so long
without being updated or maintained, it never gets done.

That was something I wanted to avoid with [SpotSquid](https://spotsquid.com){:target="_blank"} and to try to take a
long term view of things and make sure things never get to the point of no
return.

I have a few processes I've started using to keep my running SaaS app updated as
a solo developer.  I'll be going through how I keep my Jumpstart Pro app updated
while still shipping features and other changes.

### Dependabot

Dependabot updates have been a _huge_ help for me keeping things updated.  If
you don't make the time to try to stay on top of them though, they can quickly
pile up and become a distraction.

Jumpstart Pro ships with configuration for Dependabot so you'll start seeing
these pull requests start coming in shortly after pushing to github.


The current Jumpstart Pro default for how often Dependabot checks for updates is
weekly.  If this is a little optomistic for your taste, you can change the
`interval` value to `"monthly"` to only check for updates once per month.

If you want to go the opposite direction, you can also configure Dependabot to
check for updates daily.

[Dependabot Interval Options](https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/configuration-options-for-the-dependabot.yml-file#scheduleinterval){:target="_blank"}

[Dependabot Configuration Options](https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/configuration-options-for-the-dependabot.yml-file){:target="_blank"}

The three `package-ecosystems` we're telling Dependabot to watch for changes for
are `"github-actions"` for your actions you've added and configured as part of
your Build/CI process on GitHub, `"npm"` and `"bundler"`


This will check NPM for updates to the JS packages and Bundler for ruby gem
updates and open a pull request to our app bumping the versions.

#### Merging changes

<img class="position-relative mx-auto rounded w-100 shadow-lg" src="https://personal-blog-assets.s3.amazonaws.com/DependabotPullRequests.png" />

This is an exmaple of what you'll see when you view the Dependabot pull requests
in GitHub.

One thing you'll notice is we have some nice labels showing that each one of
these is a dependency update and if it's for JS or Ruby.  Nevermind the failing
builds, I haven't had time or desire to fix all of that but I think this can be
pretty close to automatic where it merges after tests pass.

One option is to merge each one of these changes individually.  This is what I
was trying at first, but it was a little time consuming and without doing a
quick check after each merge, if issues surfaced, it was hard to pinpoint
exactly where the problems were introduced. This one has _really_ burnt me with
some of the JS package updates.

That way if something goes south in the upgrade, I don't have to worry about
reverting it, I can just delete the upgrade branch and come back to it whenever
found to be a good mix of speed and sensibility

I start with the Ruby gems since that's where I have the most experience with
upgrades.

To start, I create a new branch from `main` using something like:

`upgrades/ruby/07-19-23`

This branch can be named anything but it's not uncommon to have to change gears
or run out of time in the middle of things so this is something that helps give
me more context when I come back to things.

Then I push this branch up to GitHub, update the target branch for each gem
upgrade PR to my upgrade branch `upgrades/ruby/07-19-23` and merge the changes.

After merging in the updates to the upgrade branch, I pull down the changes to
my local machine and do some quick checks to make sure everything is working.  I
have a pretty decent amount of test coverage that I also rely on to let me know
when things go south, but running things locally before merging makes sure when
you come back to things, everything starts and works.  Things like conflicts
with local versions or other gems that haven't released a patch for known issues
are all little snags you can hit that slow you down and steal precious time from
your side project.

If everything looks good locally and all my tests pass on CI, I merge those
changes and move one.

_If_ things do go south, I at least know it's an issue with some of the recent
gems and can either A) revert the changes and try to merge each gem individually
to try to find where the problems are being introduced or B) Declare upgrade
bankruptcy on this one and get try again on the next round of upgrades.  If
you're on the monthly interval, that might be a little more time than you'd like
to put things off. The importantl thing is to find what works best for _you_.

Another big perk of merging things in groups like this is those issues are
contained within the upgrade branch instead of merged into your main branch
where you feel like you have to try to frantically fix a broken deploy.

Then I repeat the same process for the JS pacakges.  I'm not pointing fingers
but the JS pacakges seem more likely to introduce breaking changes in minor
versions so I _really_ try to keep these separate and make sure everything is
working as a whole before merging.

Having a set of steps and processes for merging these updates lowers some of the
friction to upgrading and makes it a little more enticing and manageable.
Especially if you're working solo, you have to get the most out of the limited
time you have.

I would say currently, if most of the versions bumps are pretty small, it takes
about 15-20 mins to do those updates.

### Scheduled reminders and recurring tasks to bug my into it

Depending on your notifications, you probably see something when Dependabot
opens a new pull request.  However, this will _not_ let you know when there are
upates to the upstream repo (the repo you forked Jumpstart Pro from).  It's easy
to forget about this let your verions drift further and further from upstream.
The longer it goes, the more involved the upgrade process is.

As a way of gently nudging and reminding myself to upgrade I have a monthly
Slack reminder to 'Check on upgrades' which is kind of a placeholder for
dependabot updates if they're not already covered _and_ checking on the upstream
chagnes from Jumpstart Pro.

I also have a recurring Trello card that gets created once per month to at least
_check_ on the upgrades.

I don't always have the time to upgrade my application as much as I like, but at
least this way, you stay updated on what's changing.

It also helped to re-frame how I thought about these updates.  Instead of
thinking of pulling upstream changes as a chore, I started thinking about it
more along the lines of leveraging what's available to me.

Pulling these updates and staying on top of the changes allow you to take full
advantage of the GoRails team. Thinking of it that way as a time-strapped solo
founder, why _wouldn't_ I want the GoRails team porting changes that update and
enhance my app.

Thinking of it as a perk instaead of a chore made it sit a little better with
me.

### Upstream Changes

I've done quite a few of upgrades to Jumpstart at this point and this is the
process I've ended up with.  Feel free to tweak as needed but I really think
having a process or some rules for how you approach these things save your
mental bandwidth for the interesting challenges your app is tackling.  Finding a
flow and set of steps that works well for you is important to making this
something that's repeatable and not so draining.

These steps are assuming you've set up your local repo according to the
Jumpstart Pro docs here https://jumpstartrails.com/docs/upgrading

in particular `git remote rename origin jumpstart`


First, I pull the latest changes from upstream (the main JSP repo)

```bash
$ git fetch jumpstart
```

<img class="position-relative mx-auto rounded w-100 shadow-lg" src="https://personal-blog-assets.s3.amazonaws.com/JumpstartUpstreamFetch.png" />

Then, I create a branch in the same way I do for the gem updates

```bash
$ git checkout -b upgrades/july-19
```

I don't usually have other open upgrade branches while updating JSP so don't
include anything noting this is for upstream changes but that might help.


After creating my upgrade branch, I merge the jumpstart branch into my upgrade
branch and push to GitHub.

```bash
$ git merge jumpstart/main
```

If you've been updating regularly, this is usually pretty smooth. You may have
to resolve some conflicts before completing the merge.  If it's only a handful
of conflicts and/or you feel comfortable resolving conflicts once everything is
resolved and you've merged your branch you can push that branch up to GitHub to
start comparing the changes.

If it's been more than a couple of months since you've pulled updates from the
upstream version of Jumpstart Pro, you'll probably have a lot of conflicts to
resolve.  If you don't want to tackle all of those conflicts at once, this is my
process for making that a little easier.

When I need more time and more clarity to work through all the changes, I will
create a branch with the code from Jumpstart upstream main branch, push that to
GitHub, then open a pull request to an upgrade or my main branch (depends on how
brave I'm feeling).

Fetch the latest version on the Jumpstart remote and checkout the main branch

```
$ git fetch jumpstart
$ git checkout jumpstart/main
```

You'll see a notice about 'detached HEAD' which is read-only.  We need to switch
branches so we'll be able to make commits.

```
$ git switch -c jumpstart-updates-july
```

This gives us a local branch with the code we were trying to merge with `git
merge jumpstart/main`

Push the new branch to GitHub

```
$ git push origin jumpstart-updates-july
```

After pushing, I open a pull request on GitHub from the upgrade branch to the
main branch.  This gives me a nice visual diff of all the changes in a single
spot.  I've found this is much easier for me to digest than trying to do
everything locally in Git.

It's also helpful to mark the changes you're comfortable with as 'Viewed' to
help remind you of the files you've already reviewed if (and when) you have to
come back to this and finish up.


(Screenshots of Diffs and collapsed changes)
<img class="position-relative mx-auto rounded w-100 shadow-lg" src="https://personal-blog-assets.s3.amazonaws.com/JumpstartProUpstreamDiff.png" />


This gives you a nice visual to see all the changes.  After reviewing some of
the changes you may want to do something like "just accept all the changes in
lib" and "ignore the changes in app/views".  That's really up to you and how you
prefer to do things.  Resolving conflicts is out of scope for this article but
the `--ours` and `--theirs` flags for git can be a big help. You can read some more info on those [here](https://howchoo.com/git/git-merge-conflicts-rebase-ours-theirs){:target="_blank"}
For me, I'm not actively working or making changes to
`lib` so feel ok marking those as revivewed and generally ignore all the changes
to my views and cherry-pick or copy-paste any changes I'd like to include.

As mentioned before, keeping this potentially hairy upgrade within it's own
branch giving you plenty of time to review and test prior to merging allows you
to skip all together or pick back up where you left off whenever you run out of
time.  Merging the upgrade branch into your main branch this way also gives you
the option to revert those changes with a single click.

The worst thing as a solo developer is being stuck trying to get my app running
whenever I have time to work on it.  Quarantining your upgrades lets you keep
rolling as needed.

After running the upgrade branch locally and making sure everything seems to be
in order, after feeling pretty comfortable about the changes I'll merge and
depoly to my staging server.

If everything looks good, or at least nothing looks bad, I deploy those changes
to production.

### Finishing up

I wouldn't say I _love_ all of the upgrading I've been doing but I will say it's
been a huge help in keeping my apps up to date and finding ways to help myself
and others keep theirs updated as well.

I've been working on building apps with Jumpstart for about 2 years now both
personally and professionally and would love to talk to you about how I might be
able to help (at some point, no availability right now)

