---
layout: post
title:  Route Constraints with Rails
date:   2022-05-17 16:54:46 -0500
category: ruby
excerpt: "How I tackled some pesky interference in my routes after adding a url
shortener"
author: cody
---
I was having some problems with a URL shortenrer gem I added recently.  The
shortener kept getting requested when viewing routes in my static controller.
The easiest solution would have been to just mount the enginge under a prefix,
something like `/srt`, which is exactly what I ended up doing, but my first
attempt at solving the issue was using route constraints.

That was something that I'm pretty sure I had heard of, but never had any
experience so thought it would be a cool problem to tackle.

There are two types of constraints, one that's a little more simple, and one
that is it's own class.  The latter is what I eneded up going with.

The  one with the class, and [rpbably the other one as well, takes in a
`request` objecy MORE INFO on request objects here


My first pass had me adding the routes of my static pages to an array, and check
the `request.path` to see if it was included in the request.

That worked just fine, but after I added a new static page, I got another test
fail for 404 (what happens when the shortener can't find an object)

I wanted to see if there was a way to make that check dynamic and keep me from
having to update manually, usually after seeing a spec fail and having to
refresh my memory.



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

```ruby
# config/routes.rb
require 'app/services/ShortenerRouteConstraint.rb'

# get "/:id" => "shortener/shortened_urls#show", constraints: ShortenerRouteConstraint.new
```

After checking all the pages in the static directoy, I add `/admin`, `/404`, and
`/500` to the constraint for the other routes it was having a conflict with.

Like I mentioned earlier, shortly after implementing, I ended up going a
different route, but thought it was an interesting challenge adding the
constraint and making the checks dynamic and wanted to share.
