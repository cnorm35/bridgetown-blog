---
layout: post
title:  Using Concerns with Ruby on Rails
date:   2023-07-23 16:54:46 -0500
category: ruby
excerpt: "Looking at what are Rails concerns and how and why to use them"
author: cody
---
For the past few months, I've been holding standups and office hours with a top-notch group of
early career developers.  After everyone gives their update, we take some time
and try to go into more detail about one of the issues they're having.  Topics
have ranged from database migrations and when to use the `up` and `down` methods over
`change`, helper methods, how and what to test and plenty more.

On one of these in-depth conversations, we found ourselves tracing down a method
definition that led us to a module using `ActiveSupport::Concern`, as I'll refer
to it from here on out, a Concern.

I've used Concerns a ton over the years but it's a topic I've never had to look
into more than just a quick skim of the docs to check the syntax or make a few
changes to an existing Concern.

I realized that while I knew how to use Concerns, I didn't know enough to be
able to explain all of the details without re-visiting the documentation.

This is something that comes up a lot in these mentorship sessions.  I have a _decent_
idea how something works. When I don't, I can usually make an educated guess
based on how other things work and are set up in Rails.

When working with these early career devs, I always try to highlight when I
don't know something, which is well...a lot. One of the things I try to stress
is you don't have to know or memorize everything about Rails (or any other
framework for that matter).

If you know enough to find what you're looking for
and can read over some documentation or source code and get what you need,
there's no point in memorizing a ton of methods and other documentation. You'll start to learn and memorize the right things the more you work
and figure out the things you're having to look up consistently.

I thought this was a great opportunity to refresh myself on some of the details
around Concerns in Rails, why they're useful and how to implement them.

### What is a Concern?

A [Concern](https://api.rubyonrails.org/classes/ActiveSupport/Concern.html){:target=>"_blank"} is a Module with added methods that makes it easier to define instance and class methods that will be evaluated in the context of the Class they're included into.

From the [Rails
Docs](https://api.rubyonrails.org/classes/ActiveSupport/Concern.html) we can
look at an example of a Concern, and what an equivalent module would look like
without the extras provided by `ActiveSupport::Concern`

```ruby
# example using ActiveSupport::Concern
require "active_support/concern"

module M
  extend ActiveSupport::Concern

  included do
    scope :disabled, -> { where(disabled: true) }
  end

  class_methods do
    ...
  end
end
```

```ruby
# plain ruby module version
module M
  def self.included(base)
    base.extend ClassMethods
    base.class_eval do
      scope :disabled, -> { where(disabled: true) }
    end
  end

  module ClassMethods
    ...
  end
end

```


Not only does `ActiveSupport::Concern` give us a cleaner interface to dynamically
define methods into the classes they're included in, it can also resolve module
dependencies.

```ruby
module Foo
  def self.included(base)
    base.class_eval do
      def self.method_injected_by_foo
        ...
      end
    end
  end
end

module Bar
  def self.included(base)
    base.method_injected_by_foo
  end
end

class Host
  include Foo # We need to include this dependency for Bar
  include Bar # Bar is the module that Host really needs
end
```

```ruby
module Bar
  include Foo
  def self.included(base)
    base.method_injected_by_foo
  end
end

class Host
  include Bar
end
```

### Why Concerns?

Concerns make it easy to share code across different Classes and Modules. Concerns allow us
to define either class methods or instance methods on the objects they will be
included to.

Looking at the first example above, we see the Concern is defining a new scope
called `disabled` 


```ruby
# MyConcern
included do
  scope :disabled, -> { where(disabled: true) }
end
```

The code within the `included` block will be added to any Class it's included
in.

Say we have a `User` model with a `disabled: boolean` column

```ruby
class User < ApplicationRecord
  include MyConcern
end

```

Pending that all the data is set up correctly, we would now be able to use the
scope defined in the Concern

```
disabled_users = User.disabled
# returns collection of Users
```

<!-- Now, when we need that same functionality, we don't have to re-write or -->
<!-- duplicate that method, we can just include the Concern -->

Let's say we would like to have that same `disabled` scope available in an
`Account` class.  Instead of defining the scope in this class, we can just
include the Concern.

```ruby
class Account < ApplicationRecord
  include MyConcern
end
```

```ruby
disabled_accounts = Account.disabled
# returns collection of Accounts
```

This gives us a clean and powerful way to define logic that we'd like to share
across different objects.

We're also not limited to class methods, we can also define instance methods.
Assuming both the `User` and `Account` models from above have a `name` field
that's a string.

Let's say we want to format the `name` field a specific way to create a filename.

```ruby
module FileNames
  include ActiveSupport::Concern

  def to_filename
    ActiveSupport::Inflector.parameterize(name, separator: '_')
  end
end
```

```ruby
class User < ApplicationRecord
  include FileNames
end

class Account < ApplicationRecord
  include FileNames
end
```

```ruby
user.to_filename
"cody_norman"
account.to_filename
"my_business_account"
```

If you find yourself including the same concern many times, you may be able to include it
in a parent class like ApplicationController or ApplicationRecord to make that
functionality available across any child class.

### Where to put your concerns?

If you look inside your `app/controllers` and `app/models` directories, both
have a `concerns` directory.

If I'm not mistaken, there isn't any Rails magic that requires them to be in a
specific spot, those directories are there for organization and clarity.

Without overthinking it too much, if your Concern will be added into a
controller, it should be in the `app/controllers/concerns` directory and the
same applies for models.

If you're working with something that interacts with both Models and
Controllers and don't want to break into smaller chunks, consider placing
somewhere like `lib`.

### Custom Errors

One thing I like to add to Concerns is a custom error class I raise if there are
any errors.  Raising the specific error allows error handling to be more
specific and explicit.  Seeing something like `NameIsNilError` is a lot more
intuitive than something like a `method_missing` error you have to dig through
the stack trace for.

```ruby
module FileNames
  include ActiveSupport::Concern
  class NameIsNilError < StandardError; end

  def to_filename
    raise NameIsNilError, "unable to generate filename, name is nil" unless name
    ActiveSupport::Inflector.parameterize(name, separator: '_')
  end
end
```

### Testing

There are a few different ways to test Concerns. Some people prefer doing
something like a shared context for the functionality the Concern adds, others
prefer testing the methods directly in the objects they're included in. 

There's also the option of creating a dummy class to include your Concern into and testing
that class.  This is the method I go for most of the time. In the same way
concerns help keep duplication to a minimum, I like this option since I'm not
continuously adding testing to new objects

I think this is especially a good option for concerns included at a higher level
like the `ApplicationController`
<!-- make a note about custom errors then give a quick example on testing -->

If we have a `Users::TimeZone` concern included to our ApplicationController to
check the user's timezone, this is an example of including into a dummy class
and testing the dummy class

```ruby
#controller_spec.rb

# create a dummy class to include the concern we're testing
class MockController < ApplicationController
  include Users::TimeZone
end

RSpec.describe MockController, type: :controller do
  let(:browser_time_zone) { "America/Los_Angeles" }
  # setup for cookies

  before do
    allow(controller).to receive(:cookies).and_return(browser_time_zone: browser_time_zone)
  end

  describe "#browser_time_zone" do
    expect(controller.browser_time_zone).to "America/Los_Angeles"
  end
end

```

I simplified that example a bit to make it more readable to focus on creating a
class within the spec, including our subject Concern to that class and testing
the Concern's functionality.

### Closing

Concerns are a great way to keep your code DRY.  When you find yourself
re-writing or repeating similar methods across different classes, think about moving that logic to a concern.  Hopefully, this post has
prepared you to better handle the next time to have to update existing concerns
or find an opportunity to add a new one.
