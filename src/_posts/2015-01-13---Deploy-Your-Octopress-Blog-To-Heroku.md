---
title: "Deploy Your Octopress Blog To Heroku"
date: "2015-01-13T22:56:49.169Z"
template: post
draft: false
categories: "Ruby"
tags:
  - "Ruby"
description: "Walkthrough on how I deployed my first static site with Octopress to Heroku"
---
You may have noticed that my site's name is codebycodes, but I have yet to post any code on here.  Good job on calling me out.  So here it is.  This post will walk you through setting up a new blog with the Octopress blogging framework and deploy to Heroku.  This is the same setup that this blog is currently running on.

So first off, what exactly _is_ Octopress.  Octopress is billed as a blogging framework for hackers.  It's a little different that some of the other blogging frameworks you may have had some experience with.<!--more-->

The first is it's built with Ruby on top of another framework called Jekyll.  The second, is there is no database.  You create your posts using the [markdown](http://en.wikipedia.org/wiki/Markdown) language and Ruby (with the help of Octopress) generates static HTML pages.  This make it pretty easy to set up and very fast since everything is a static page.

To get started, fire up your command line and clone a copy of the Octopress repository.

```
$ git clone git://github.com/imathis/octopress.git my_blog
```
Next, navigate into the folder that was created when you cloned the Octopress code.

```
$ cd my_blog
```

Since Octopress is built off Ruby, lets check the current version of Ruby installed.

```
$ ruby -v
```

If it's not at least `ruby 1.9.3` check out the documentation to upgrading your ruby version with RVM [here](http://octopress.org/docs/setup/rvm/)

So now, lets install the gems 

```
$ gem install bundler
$ bundle install
$ rake install
```
At this point, you should be able to see the basic template for your site.  Running 
```
$ rake preview
```
will generate the required files for you site and start a simple web server.  Check out localhost:4000 to see if your site is up and running.  It should look something like this

{% img left /images/Octopress_Screenshot.png 800 350 'image' 'images' %}

Now we need to remove `public` and `Gemfile.lock` from our `.gitignore` file.  We'll need this to add generated content to Heroku.

There are also a number of themes you can download and use for your Octopress site, or you can create your own.  You can find a good list of available themes [here](https://github.com/imathis/octopress/wiki/3rd-Party-Octopress-Themes).  There are pretty good instructions for downloading and installing a new theme for your blog.

Once that's take care of, let's head to `_config.yml` and start customizing the settings

Main settings:
```yaml
url: http://www.codebycodes.com
title: Code by Codes
subtitle: "Codes, rants and musings"
author: Cody Norman
```
If you'd like to connect some of your external accounts, scroll down and check out some of the 3rd party services you can connect.

To add or remove services, add or remove the service in the `default_asides` section of the `._config.yml` file.

```yaml
default_asides: [asides/recent_posts.html, asides/github.html]
```
I think at one point, there was a default Twitter aside ready to go.  But now we'll have to make our own.  Since this seems like it'd be one of the more popular things to have.  I'll show you how to add it.

First, add Twitter to your list of default asides.

```yaml
default_asides: [asides/recent_posts.html, asides/github.html, asides/twitter.html]
```

Now, let's head to Twitter to generate the code we need.  Log into your Twitter account and click on `Settings`.  Once you're there click on the `Widgets` tab.  You can update some of the options if you'd like.  There is a nice preview on the side so you can play around with your configuration to get it set up how you like.  Now that you have all your settings squared away, click on the `Create Widget` button and copy the generated code.

Create the file `source/_includes/asides/twitter.html`  Once you have the new file open paste in the code you got for the Twitter widget.

```html
	<section>
		<h1>Twitter</h1>
		<!-- PASTE THE CODE HERE! -->
	</section>

```

With some bells and whistles, it's time to create our first post.  We create all our new posts from the command line.  With your command line open in your blog's directory, let's create a new post

```
$ rake new_post["your title goes here"]
# Creates source/_posts/2015-01-14-your-title-goes-here.markdown
```

The `new_post` command expects a naturally written title so don't worry about underscores, just write out the title as your normally would.  This filename will determine the url for the post.
`yourblog.com/blog/2015-01-14-your-title-goes-here/index.html`.

With our first post created, open up the file that was created and notice the top lines of the file.  This is the [yaml front matter](http://jekyllrb.com/docs/frontmatter/).  This is where you can set the predefined variables or add your own.

```yaml
---
layout: post
title: "Deploy your Octopress blog to Heroku"
date: 2015-01-13 18:08:38 -0500
comments: true
categories: octopress, heroku
published: false
---
```

All of this info is going to be used by Octopress to process the posts and pages.  You're going to be writing your post in the [Markdown](http://en.wikipedia.org/wiki/Markdown) language.  This is the same language you would see in the documentation on github project.  This format makes it much easier to share code snippets.  The syntax will take some getting used to, so you can find a pretty good cheatsheet [here](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet).

After you have some info in your post, let's check out how it looks.  First, let's run `rake generate` to create our pages into static html

```
$ rake generate
```
Once all our static files are created, running `rake preview` will start up a web server and let us take our new blog for a test drive.

```
$ rake preview
```

Now we have a fully functioning blog, all we need to do now is deploy it somewhere.  This post will cover how to deploy your blog to Heroku.  You can also deploy your site to Github Pages.  Each user can have a free page on github (heroku is free also).  I chose Heroku because I normally have a couple of test or demo apps running anyway (you have up to 5 for free) so it makes it a little easier for me to manage.  If you'd like to look into Github pages more, Octopress has some great [documentation](http://octopress.org/docs/deploying/github/)

In your terminal, install the `heroku` gem

```
$ gem install heroku
```
If you've never deployed to Heroku, head over to their site and create an account.  Once you have your account, in your command line run `heroku create`

```
$ heroku create
```

It will ask for your account credentials and upload your public SSH key.  If you've never created a public SSH key, [Github](https://help.github.com/articles/set-up-git/) has a really great guide to getting it setup.

Once of my favorite features to Heroku is it's as easy to deploy as pushing code to Github.

Create a new app on Heroku and add a git remote named 'heroku'

```
$ git config branch.master.remote heroku
```
If you haven't done so yet, make sure that you've edited your `.gitignore` file be removing `public` and `Gemfile.lock`.  Again, this will let you add your generated content to Heroku.

With your remote branch set up, we just need to commit and push as we would any normal git repository.

```
$ rake generate
$ git add .
$ git commit -m "updated my awesome site"
$ git push heroku master
```

Once that's complete, you can either head to the url that Heroku gave to you or run the open command from your command line and open it automatically.

```
$ heroku open
```

And that's it!  You now have your own blog up and running on Heroku.  If your having any issues, [Octopress](http://octopress.org/docs/) has some really helpful documentation and so does [Heroku](https://devcenter.hero7ku.com/start).  If you dont have any luck there, leave a comment and we'll see if we can get it sorted out.
