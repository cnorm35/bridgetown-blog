---
layout: post
title: Creating Rails Application Templates
date:   2015-08-17 16:54:46 -0500
category: ruby
excerpt: "Why choose a template over just having a full application you can clone from github as a fresh starting point? I think Rails templates are a much more powerful and versatile option.  Rails templates are built on Thor which is probably the best feature in Rails you’ve never heard about."
author: cody
---
_Update 2022: Rails templates have came a long way since this was originaly
created. These days I would look into something like [Rails
Bytes](https://railsbytes.com/)_

Programmers are lazy.  This is probably one of the greatest things about us.  Anything that takes slightly longer than we think it should, and you start thinking “Hmmmmm……I wonder if I could fix that” or “I bet I could make that better".

I don’t know about you, but I’m the type that can get easily distracted by shiny new things to try out, whether it’s a new framework, gem, pattern, or fancy bourbon (I mean, we can’t program all the time).

I started looking into Rails application templates to try to alleviate these problems.  If I want to try out a new feature, follow along with a tutorial, or just start tinkering with a new project, it takes me a little while to get everything set up with my preferences.  Things like setting up devise, bootstrap, getting flash messages to play nicely with bootstrap and any other configs I think will help with whatever project I’m kicking off.

Why choose a template over just having a full application you can clone from github as a fresh starting point? I think Rails templates are a much more powerful and versatile option.  Rails templates are built on Thor which is probably the best feature in Rails you’ve never heard about.  [Thor](http://whatisthor.com/) is described as:

_... a toolkit for building powerful command-line interfaces. It is used in Bundler, Vagrant, Rails and others._

Ever notice all the files that are created and how you don’t have to run bundle install after you create a new rails app?  You have Thor to thank for that.

Having the ability to run commands and edit files is a huge advantage.  It’s also an easy way to stick to best practices with less effort.  How?  You can do things like set up guard, r-spec, and even create a git repo and make an initial commit, all without having to write any code...well, except for the template.

I’ll admit, sometimes I can be a little sloppy with having tests in my personal apps.  If I’m pressed for time, I’ll normally choose cramming as many features as possible (the F-it Ship it mantra) than tests.  I think this helps combat that by having my test suite all set up and ready to go.

If you want the TL;DR version, you can create a new rails app from the template by running:

```
  $ rails new your_project_name -m https://github.com/cnorm35/starter_template/blob/master/test_template.rb
```

Or you can see the entire template [here](https://github.com/cnorm35/starter_template/blob/master/test_template.rb).

Let's break apart the pieces and see what everything is doing.  Check out the [RailsGuides for Applicaiton Templates](http://guides.rubyonrails.org/rails_application_templates.html) for some more info, and a detailed rundown of the available methods.

The first thing we'll do it overwrite the `source_path` method in Thor with the location of our template.  By doing this, Thor will accept relative file paths to the location of our template.

``` ruby
def source_paths
  Array(super)
  [File.expand_path(File.dirname(__FILE__))]
end
```

The next step, we set up the Gemfile.  You can either remove and add gems to your existing Gemfile individually, or you can remove the Gemfile all together and start from scratch.  I prefer to start from scratch to I dont have to remove all the comments Rails adds automatically afterwards.  Just be sure to include a source at the top of the Gemfile.

```ruby
remove_file "Gemfile"
run "touch Gemfile"
add_source 'https://rubygems.org'
gem 'rails', '4.2.1'
gem 'sqlite3'
gem 'uglifier'
gem 'coffee-rails'
gem 'jquery-rails'
gem 'turbolinks'
gem 'thin'
gem 'devise'
gem 'bootstrap-sass'
gem 'sass-rails'
gem 'simple_form'
gem 'sdoc', group: :doc
gem_group :development, :test do
  gem 'spring'
  gem 'quiet_assets'
  gem 'pry-rails'
  gem 'byebug'
  gem 'awesome_print'
  gem 'better_errors'
  gem 'binding_of_caller'
end

gem_group :test do
  gem 'guard-rspec'
  gem 'rspec-rails'
end
```

Since we'll be using Rspec, after bundle, we can remove the `test` directory with the `after_bundle` callback.
```ruby
after_bundle do
  remove_dir 'test'
end
```

Then we'll change the README to markdown.
```ruby
remove_file 'README.rdoc'
create_file 'README.md' do <<-TEXT
  #Markdown Stuff!
  Created with the help of Rails application templates
  TEXT
end
```

Here's where things start to get a little more interesting.  The Rails template API provide a `generate` [method](http://guides.rubyonrails.org/rails_application_templates.html#generate-what-args).  The `generate` method works pretty much the same as if you were running it from your console.  You can use it to generate scaffolds, or even included generator for gems.

```ruby
generate 'rspec:install'
run 'guard init'

generate 'devise:install'
generate 'devise User'
generate 'devise:views'

rake 'db:migrate'
```


BOOM!  Now, when we create a new app from the template, we have Rspec installed and Devise already set up with a `User` model and the db migrated.

Now, since Devise will direct you to the root path after creating a new registration or signing in, lets create a new controller and home page to redirect to.

```ruby
generate(:controller, "pages home")

remove_file 'app/views/pages/home.html.erb'

create_file 'app/views/pages/home.html.erb' do <<-TEXT
<div class="jumbotron center">
  <h1>Tempaltes, Whoop Whoop!</h1>
  <p>Template created with the help of <a href="http://codebycodes.com">CodeByCodes.</a>/p>
  <h4><%= link_to 'Sign Up', new_user_registration_path %></h4>
</div>
TEXT
end

route "root 'pages#home'"
```

You may have noticed, we're using some Bootstrap classes.  Let's go set up the app to use Bootstrap.  Note:  I have some extra style rules added in so Devise and Bootstrap
will play together nicely.  I'll go over that in more details later.

```ruby
create_file 'app/assets/stylesheets/customizations.css.scss' do <<-TEXT
  @import "bootstrap-sprockets";
  @import "bootstrap";
  .center {
    text-align: center;
  }
  /*for Devise Rememver me box*/
  #user_remember_me {
    margin-left: 0px;
  }
  /*flash*/
  .alert-error {
      background-color: #f2dede;
      border-color: #eed3d7;
      color: #b94a48;
      text-align: left;
   }
  .alert-alert {
      background-color: #f2dede;
      border-color: #eed3d7;
      color: #b94a48;
      text-align: left;
   }
  .alert-success {
      background-color: #dff0d8;
      border-color: #d6e9c6;
      color: #468847;
      text-align: left;
   }
  .alert-notice {
      background-color: #dff0d8;
      border-color: #d6e9c6;
      color: #468847;
      text-align: left;
   }
  TEXT
end
```

Now to add it to `application.js`.  Thor provides the really handy methods `before` and `after` to target where you would like something inserted.

```ruby
insert_into_file('app/assets/javascripts/application.js', "//= require bootstrap-sprockets\n", :before => /\/\/= require_tree ./)
```
In other words, find `require_tree` in the file, and right before it, insert `"//= require bootstrap-sprockets"` with a new line after.

I'll skip over some of the other setups for the views, it's jsut some view files with a little styling applied to it.  Again, the complete templete is available [here.](https://github.com/cnorm35/starter_template/blob/master/test_template.rb)

One thing I do want to address, is setting up flash messages with Devise.  Devise ships with some less-than flattering flash messages and updating them to use Bootstrap isn't always straightforward.  Luckily, there's a page on the [Devise wiki](https://github.com/plataformatec/devise/wiki/How-To:-Integrate-I18n-Flash-Messages-with-Devise-and-Bootstrap), so you dont have to dig though stack-overflow posts anymore.  The next few blocks will just be setting that up.

```ruby
create_file 'app/views/layouts/_messages.html.erb' do <<-HTML
  <% flash.each do |key, value| %>
  <div class="alert alert-<%= key %>">
    <a href="#" data-dismiss="alert" class="close">×</a>
      <ul>
        <li>
          <%= value %>
        </li>
      </ul>
  </div>
<% end %>
HTML
end

insert_into_file 'app/views/layouts/application.html.erb',
  "\n<%= render 'layouts/header' %>\n <%= render 'layouts/messages'%>", :after => /<body>/
```

Now, create the helper file
```ruby
create_file 'app/helpers/devise_helper.rb' do <<-TEXT
  module DeviseHelper
    def devise_error_messages!
      return '' if resource.errors.empty?
      messages = resource.errors.full_messages.map { |msg| content_tag(:li, msg) }.join
      html = <<-HTML
      <div class="alert alert-error alert-danger"> <button type="button"
      class="close" data-dismiss="alert">x</button>
        { messages }
      </div>
      HTML
      html.html_safe
    end
  end
TEXT
end
```

Here was my biggest _Gotcha_ of the day.  You may notice in the above block it's just `{ messages }` instead of `#{ messages }`.  When running the template, rails kept complaining
`messages` was undefined.  I could have extractred that out into a seperate file, but I wanted to try to keep everying in a single file.  Also, I try to not focus too much on over optimizing at the beginning and stick with "Done is Better Than Perfect".

Since Thor provides `:before` and `:after`, my solution was to find `{ messages }` and insert the `#` directly before it to make the string interpolation work properly.

```
insert_into_file('app/helpers/devise_helper.rb', '#', :before => /{ messages }/)
```

So there you have it, a working Rails template.  Let me know in the comments if there's anything I missed or some things you think I should have added.  Since this was the first template I've created, it's still a work in progress.  I'll try to keep it updated as my setup evolves over time.
