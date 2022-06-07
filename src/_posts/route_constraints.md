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

According to the Rails Guides:

> _You can also constrain a route based on any method on the Request object that returns a String._

There are a few different options for route constraints.  And originally went
with using a lambda within the route definition but eventually moved to the
using a class.

[Rails Guides on Advanced Route Constraints](https://guides.rubyonrails.org/routing.html#advanced-constraints)

lambda route constraint example

```ruby
  get '*path', to: 'restricted_list#index',
    constraints: lambda { |request| RestrictedList.retrieve_ips.include?(request.remote_ip) }
```

class route constrain example

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

My first pass had me adding the routes of my static pages to an array, and check
the `request.path` to see if it was included in the request.

That worked just fine, but after I added a new static page, I got another test
fail for 404 (what happens when the shortener can't find an object).

I wanted to see if there was a way to make that check dynamic and keep me from
having to update manually, usually after seeing a spec fail and having to
refresh my memory.

With our class for constraining the routes, we need to update the route
definition to use the constraint.

```ruby
# config/routes.rb
require 'app/services/ShortenerRouteConstraint.rb'

get "/:id" => "shortener/shortened_urls#show", constraints: ShortenerRouteConstraint.new
```

_Note: I really didn't know where to best place to put that class, so just
defaulted to putting it in my `app/services` directory and requiring that file
at the top of my `config/routes.rb`_

For finding paths to whitelist, one option would be to have an array of the
static pages and add a new entry whenever I created a new page.  Checkng the
routes against a dynamic list of view files is absolutely overkill, but really
wanted to see if I could find a way to set and forget.

A few minutes of skimming through the Ruby docs and found what I needed.

```ruby
  # dynamically pull all static pages
  static_page_paths = Dir.new("app/views/static").children.map do |f|
    "/#{File.basename(f, ".html.erb")}"
  end
```

This creates a new Dir ADD LINK object for `app/views/static` directory.  We access it's
children, use map to return a new array with a format that matches our path for
the route constraint

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

This will pull the name of any view ending in `.html.erb`, including partials in the `app/views/static` directory

After checking all the pages in the static directoy, I add `/admin`, `/404`, and
`/500` to the constraint for the other routes it was having a conflict with.

Like I mentioned earlier, shortly after implementing, I ended up going a
different route, but thought it was an interesting challenge adding the
constraint and making the checks dynamic and wanted to share.
