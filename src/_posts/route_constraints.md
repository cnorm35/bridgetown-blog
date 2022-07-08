---
layout: post
title:  Route Constraints with Rails
date:   2022-07-07 16:54:46 -0500
category: ruby
excerpt: "How I tackled some pesky interference in my routes after adding a url
shortener"
author: cody
---
Recently, I was working on a feature sending sms messages with appointment
links. The links ended up taking up most of the message body so I wanted to look
into some options to shorten the link to conserve space in the body of the
message.

Instead of using something like Bitly, I wanted to see if there were any simple
options for something that would remain within the app.
I did some googling for link shortener gems and tried a couple of the first ones
that popped up.

I was able to take my links and create a shortened version that gets sent out in
the sms confirmation pretty easily. Everything seemed to be going fine until I
noticed some test failures. Some of my static pages were returning 404.
 After a bit of digging, I noticed that all my static pages, like
`/about` and `/contact` were being intercepted by the shortener.

When it didn't find a shortened link with an id of `about` it was returning a 404 
and breaking my static pages.

The easiest solution would have been to just mount the shorteners rails engine under a prefix,
something like `/srt`.

This is what I ended up with but wanted to attempt
to solve the issue using route constraints.

Route constraints are something I've heard of, but have never had cause to use.
I thought this would be a good chance to take a stab at routing
constraints and get some more experience with them.

There are a few different options for route constraints.  Some of the advanced
options are using a lambda in the route definition and one that's a little more
involved that uses a class for the route constraint.

My first approach was using a lambda within the route definition but eventually 
moved to using a class.

[Rails Guides on Advanced Route Constraints](https://guides.rubyonrails.org/routing.html#advanced-constraints)

lambda route constraint example:

```ruby
  get '*path', to: 'restricted_list#index',
    constraints: lambda { |request| RestrictedList.retrieve_ips.include?(request.remote_ip) }
```

class route constrain example:

```ruby
class ShortenerRouteConstraint
  def matches?(request)
    # if request.path is not in static routes it should be a short link
    static_page_routes.exclude?(request.path)
  end

  private

  def static_page_routes
    # dynamically pull all static pages + admin, 404, and errors
    static_page_paths = Dir.new("app/views/static").children.map do |f|
      "/#{File.basename(f, ".html.erb")}"
    end
    static_page_paths << "/admin"
    static_page_paths << "/404"
    static_page_paths << "/500"
  end
end
```

My first pass had me adding the routes of my static pages to an array, and
checking the `request.path` to see if it was included in the request.

That worked just fine, but after I added a new static page, I got another test
fail for 404 errors (this happens when the shortener can't find an object).

I wanted to see if there was a way to make that check dynamic and keep from
updating manually, usually after seeing a spec fail and having to
refresh my memory on what's happening.

I created a route constrain class matching the one above. After having the route
constraint class, the next step was to update the routes to use that constraint
class.

```ruby
# config/routes.rb
require 'app/services/ShortenerRouteConstraint.rb'

get "/:id" => "shortener/shortened_urls#show", constraints: ShortenerRouteConstraint.new
```

_Note: I really didn't know the best place to put that class, so just
defaulted to putting it in my `app/services` directory and requiring that file
at the top of my `config/routes.rb`_

Checking the routes against a dynamic list of view files is absolutely overkill, 
but wanted to see if I could find a way to set and forget.

After a few minutes skimming through the Ruby docs and found what I needed.

```ruby
  # dynamically pull all static pages
  static_page_paths = Dir.new("app/views/static").children.map do |f|
    "/#{File.basename(f, ".html.erb")}"
  end
```

This creates a new [Dir](https://ruby-doc.org/core-3.1.2/Dir.html) object for the
`app/views/static` directory.  We access its children, and use `map` to return a new
array with a format that matches our path for the route constraint.

This will pull the name of any view ending in `.html.erb`, including partials, 
in the `app/views/static` directory.

The output looks something like

```ruby
["/index",
 "/contact",
 "/success",
 "/privacy",
 "/pricing",
 "/about",
 "/terms"]
```


After checking all the pages in the static directory, I add `/admin`, `/404`, and
`/500` to the constraint for the other routes outside of my static pages it was 
having conflicts with.

Now, whenever I add a new page to my static controller, that route will be added
to the whitelist of static pages that should not be checked against shortened
routes.

Like I mentioned before, I ended up going a different route (see what I did
there?) but thought it was a good exercise in over-engineering a solution for 
the fun of it.
