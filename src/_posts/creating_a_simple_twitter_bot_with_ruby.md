---
layout: post
title: Creating A Simple Twitter Bot with Ruby
date:   2015-08-31 16:54:46 -0500
category: ruby
excerpt: "Let me preface this by saying I'm not always the biggest fan of Twitter bots.  With that said, they can be useful and fun to build.  Here is a quick rundown of creating a simple Twitter bot in Ruby."
author: cody
---
Let me preface this by saying I'm not always the biggest fan of Twitter bots.  With that said, they can be useful and fun to build.  Here is a quick rundown of creating a simple Twitter bot in Ruby.



First off, you will need to have an existing Twitter account. If you'd like to create a new account just for the bot you'll also need a phone number.  The easiest way to take care of this is to create a new google voice number if you've already used your phone number for your primary twitter account.  Head over to https://apps.twitter.com/ to create a new app that can communicate with Twitter's API.

It requires the app's Name, Description, and URL.  Fill out the info.  You can use something like "http://www.example.com" to get started with and update it later. Also, the Name must be unique so something like 'Test' is probably going to be taken.

Fill out the required info and create your new app.  Once that's done, you'll be taken to the Application details page.  Head over to the "Key and Access Tokens" section next.  This is where you'll find your Consumer Key and Consumer Secret Keys.  There's one more step that we need to do before we're going to be ready to go.  Scroll down to the bottom to "Token Actions" and create a new access token.  This will create the new "Access Token" and "Access Token Secret" needed to connect out bot to Twitter's API.

img here

Now that we have all of keys and tokens situated, we're ready to write some code.

This post is meant to be a quick intro to automating actions using the Twitter gem.  It will be a single file and ran from the command line so feel free to create a Gemfile, and set up a directory structure if you like.

Let's create a directory to keep our bot files in

`$ mkdir TwitterBot`

`$ cd TwitterBot`

`$ touch bot.rb`

Now that we have a directory set up and a `bot.rb` file created, let's install the Twitter gem:

`$ gem install twitter`

Time to write some code, open the `bot.rb` we created and let's get to work.

```ruby
#!/usr/bin/env ruby

client = Twitter::REST::Client.new do |config|
  config.consumer_key        = "YOUR_CONSUMER_KEY"
  config.consumer_secret     = "YOUR_CONSUMER_SECRET"
  config.access_token        = "YOUR_ACCESS_TOKEN"
  config.access_token_secret = "YOUR_ACCESS_SECRET"
end
```

First, I'd like to point out the first line at the top `#!/usr/bin/env ruby`.  Even though it has a `#` at the beginning, this is not a comment.  It's called a [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix).  This tells the program loader what command to use to run the program.  In our example, it will use the [env](http://ss64.com/bash/env.html) command to figure out where the ruby interpreter is to run our program.

Next, we create a client to communicate with the Twitter API.  Fill in the config variables with your information from the Twitter app settings page.  DO NOT CHECK YOUR KEYS INTO VERSION CONTROL!  If you plan on uploading this to GitHub or anywhere else besides your computer, [create Environment Variables](http://www.schrodinger.com/kb/1842) for your key information.

Now that we have everything connected, you can send your first tweet.  Add this line below the block where you created the `client` variable

```ruby
client.update('Tweet from the command line!')
```

Check the your Twitter account you used to register the app with, you should see your first tweet from the command line.  Being able to send a tweet from the command line is not terribly useful so let's look into something a little more interesting.  On the gem's Github page, there are a couple useful [examples](https://github.com/sferik/twitter/tree/master/examples) to get you started.  To really get an idea of the available options, you'll need to go though the [docs](http://www.rubydoc.info/gems/twitter).

First, let's pull all the recent tweets for our search criteria to make sure everything is working:


```ruby
client.search("#ruby").take(50).each do |tweet|
  puts tweet.text
end
```
This example should be pretty self explanatory, but if you're not sure what we're doing here, we're searching for tweets containing "#ruby", taking the first 50 results, and printing out the text of the tweet to our console.


If you see the tweets output to your console, you're good to go.  The baic format for the search method is:

```ruby
	client.search("query", {options})
```

You can find the available options in the [docs](http://www.rubydoc.info/gems/twitter/Twitter/REST/Search).

With the pope's visit to Philadelphia in about a month, let's say we want to get 50 recent tweets containing the word "pope", within a 10 mile radius of Philadelphia, we could do this:

```ruby
search_options = {
	result_type: "recent",
	geocode: "39.9525839,-75.1652215,10mi"
}

client.search("pope", search_options).take(50).each do |tweet|
	puts "#{tweet.user.screen_name}: #{tweet.text}
end
```

If you're feeling extra friendly, you can favorite, and reply to their tweet with a message.

```ruby
search_options = {
	result_type: "recent",
	geocode: "39.9525839,-75.1652215,10mi"
}

client.search("pope", search_options).take(50).each do |tweet|
	puts "#{tweet.user.screen_name}: #{tweet.text}
	client.favorite(tweet)
	client.upate("@{tweet.user.screen_name} Welcome to Philadelphia!")
end
```

You may want to limit interactions with replies and things like that to not come off as too spammy to people.

Here's a little example I came up with.  As a tribute to [Hitchbot](http://www.nbcnews.com/news/us-news/hitchhiking-robot-hitchbot-meets-demise-philadelphia-after-about-2-weeks-n402606) the lovable little robot that met it's untimely demise here in Philadelphia. Let's find all the 10 most recent tweets containing "#hitchbot", favorite the tweet and reply to the tweet with "Avenge Me..." as an account I setup called @hitchbot_ghost.  Take note, if you submit too many client actions at one time, the API can respond with an error saying it looks like it could be spam.

```ruby
search_options = {
	result_type: "recent",
	geocode: "39.9525839,-75.1652215,10mi"
}
client.search("#hitchbot", search_options).take(10).each do |tweet|
	puts "#{tweet.user.screen_name}: #{tweet.text}"
	client.favorite(tweet)
	client.update("@#{tweet.user.screen_name} Avenge me...",
				  in_reply_to_status_id: tweet.id)
end
```
And that's it! In just few lines of Ruby, we have a working bot for Twitter.  If you have any questions or get stuck let me know!



