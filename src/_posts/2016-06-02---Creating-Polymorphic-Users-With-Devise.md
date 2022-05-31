---
title: "Creating Polymorphic Users With Devise and Rails"
date: "2016-06-02T22:56:49.169Z"
template: post
draft: false
categories: "Ruby"
tags:
  - "Ruby"
description: "Creating Polymorphic Users with Devise and Rails"
---
First and foremost, if you're not familiar with Devise, or are looking
for a good place to get some good information on getting started with
Devise, this is not the post for you.  There are a ton of great
resources, including the Devise documentation for getting started.

Today were going to be talking about building different types of users
that will build off Devise.  It's a pretty common use case, the most
common of which is some type of service provider and a customer of that
service.  In this example we'll be using Provider and Customer.

Once we're done, we'll be able to sign up users as either a Provider or
Customer, sign them in after registering, and have a single login page
regardless of the type of user.

To make it easy to get started, I've created a template to generate a
new application for us to jump into.  If you want to know more about
creating templates, or how to create your own, check out my other post.
If you have any problems creating the template, or just prefer to fork
or clone for github, I'll include a link to the repo.

```
$ rails new polymorphic_users -m \
https://raw.githubusercontent.com/cnorm35/rails_templates/master/polymorphic_blog_post.rb
```

or

```
$ git clone https://github.com/cnorm35/polymorphic_post_starter.git
```

If you created new app from the template I provided, all you need to do
is run `rails server` and you're good to go. The migrations are already
taken care of and a new git repo initialized.


Once, you have the app up and running, lets take a look at the fields
Devise creates for the User model for us.

```
irb(main):001:0> user = User.new
=> #<User id: nil, email: "", encrypted_password: "", reset_password_token: nil, reset_password_sent_at: nil, remember_created_at: nil, sign_in_count: 0, current_sign_in_at: nil, last_sign_in_at: nil, current_sign_in_ip: nil, last_sign_in_ip: nil, created_at: nil, updated_at: nil>
```

`email`, `encrypted_password`, `reset_password_token`, `reset_password_sent_at`, `remember_created_at`, `sign_in_count`, `current_sign_in_at`, `last_sign_in_at`, `current_sign_in_ip`, `last_sign_in_ip`, `created_at`, `updated_at`

This gives us an idea of the structure that we're going to build on and
think about the best way to model our Users.  To keep the code DRY,
let's think about what fields both types of Users will have in common.
To keep it simple, let's just stick with `first_name` and `last_name`.  It's
a pretty reasonable assumption that every user that signs up for our
site will have a `first_name` and `last_name` so there's no point in
duplicating those fields in the `Provider` and `Customer` tables.


First, let's add our new fields to the Devise User model.

```
$ rails g migration add_names_to_users first_name last_name
```
That's going to create a migration file that looks like this

```ruby
class AddNamesToUsers < ActiveRecord::Migration
  def change
    add_column :users, :first_name, :string
    add_column :users, :last_name, :string
  end
end
```
Now that we have a new fields on the model, how do we add them to the
strong parameters for Devise? There are a couple ways to go about this.
You can check the recommended way from Devise [here](https://github.com/plataformatec/devise#strong-parameters).

But I prefer, a slightly different way.  For both ways, you'll need to
add a `before_filter` inside the `ApplicationController`.  This filter
will run before any action from a Devise controller.

```ruby
  before_action :configure_permitted_parameters, if: :devise_controller?
```

Let's write our method that will run before each Devise action now.

```ruby
  def configure_permitted_parameters
    devise_parameter_sanitizer.for(:sign_up) << [:first_name, :last_name]
    devise_parameter_sanitizer.for(:account_update) << [:first_name, :last_name]
  end
```

After that's taken care of, we can update our forms as we normally would
to add the new fields.  Here's what the new user form should look like
now

#check the formation on this, still looks crooked?

```html app/views/devise/registrations/new.html.erb
	<div class="panel panel-default">
	  <div class="panel-heading">
	    <h1>Sign up</h1>
	  </div>

	  <div class="panel-body">
      <%= form_for(resource, :as => resource_name, :url => registration_path(resource_name)) do |f| %>
        <%= devise_error_messages! %>

        <div class="form-group">
          <%= f.label :first_name %>
          <%= f.text_field :first_name, autofocus: true, class: "form-control" %>
        </div>

        <div class="form-group">
          <%= f.label :last_name %>
          <%= f.text_field :last_name, class: "form-control" %>
        </div>

        <div class="form-group">
          <%= f.label :email %>
          <%= f.email_field :email, autofocus: true, class: "form-control" %>
        </div>

        <div class="form-group">
          <%= f.label :password %>
          <%= f.password_field :password, class: "form-control" %>
        </div>

        <div class="form-group">
          <%= f.submit "Sign up", class: "btn btn-primary" %>
        </div>
      <% end %>
	  </div>

	  <div class="panel-footer">
	    <%= render "devise/shared/links" %>
	  </div>
	</div>

```
same thing goes for our `edit` page



``` html app/views/devise/registrations/edit.html.erb
	<div class="panel panel-default">
	  <div class="panel-heading">
	    <div class="panel-title">
	      <h1>Edit <%= resource_name.to_s.humanize %></h1>
	    </div>
	  </div>
	  <div class="panel-body">
      <%= form_for(resource, :as => resource_name, :url => registration_path(resource_name), :html => { :method => :put }) do |f| %>
        <%= devise_error_messages! %>

        <div class="form-group">
          <%= f.label :first_name %>
          <%= f.text_field :first_name, class: "form-control", :autofocus => true %>
        </div>

        <div class="form-group">
          <%= f.label :last_name %>
          <%= f.text_field :last_name, class: "form-control" %>
        </div>


        <div class="form-group">
          <%= f.label :email %>
          <%= f.email_field :email, class: "form-control" %>
        </div>


        <div class="form-group">
          <%= f.label :password %> <i>(leave blank if you don't want to change it)</i>
          <%= f.password_field :password, class: "form-control", :autocomplete => "off" %>
        </div>

        <div class="form-group">
          <%= f.label :current_password %> <i>(we need your current password to confirm your changes)</i>
          <%= f.password_field :current_password, class: "form-control" %>
        </div>

        <div class="form-group">
          <%= f.submit "Update", class: "btn btn-primary" %>
        </div>
      <% end %>
	  </div>
	  <div class="panel-footer">
	    <h3>Cancel my account</h3>

	    <p>Unhappy? <%= button_to "Cancel my account", registration_path(resource_name), :data => { :confirm => "Are you sure?" }, :method => :delete, class: "btn btn-warning" %></p>

	    <%= link_to "Back", :back %>
	  </div>
	</div>


```

Before we get to creating our new types of Users, we need to make one
more change to the Devise User model.

Start with a new migration.  If you're wondering why I didn't add it to
the last one, I like to keep my migrations small and concise.  This
makes it easier to roll things back if you ever need to.

```
$ rails g migration add_profile_to_users profile_id:integer profile_type
```

Let's add an index for the sake of performance.  Note that you can specify
things like creating an index when your passing in arguments to the
generator.  I always pop open my migrations to make sure I created it
correctly anyway, and usually add it in that way.

```ruby
class AddProfileToUsers < ActiveRecord::Migration
  def change
    add_column :users, :profile_id, :integer
    add_column :users, :profile_type, :string

    add_index :users, [:meta_id, :meta_type]
  end
end

```
One last tweak to the `User` model and we'll be all set to get started
on our new user types.

Open the `User` model and add our new association that's going to allow
us to link our new types of users

```ruby  app/models/user.rb
class User < ActiveRecord::Base
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable and :omniauthable
  devise :database_authenticatable, :registerable,
  :recoverable, :rememberable, :trackable, :validatable

  belongs_to :profile, polymorphic: true
end

```

Now we have our customizations to Devise complete, our next step is to
create our new types of users.  For the sake of simplicity, we're not going to be adding any fields
to the new types of users.  If you have no idea what polymorphic
relations are, take a bit to skim over the [Rails guides](http://guides.rubyonrails.org/association_basics.html#polymorphic-associations) to get
an idea.  If you don't get it right away, don't worry.  Polymorphism can be a tricky
concept, especially when first setting it up.  I've found that it
makes much more sense after you've implemented it and had a chance to
play around with it a bit.

If you remember, our two types of users are going to be `Provider`s and
`Customer`s.  Since both of the models are going to be pretty empty,
let's create them both at once

```
$ rails g model Customer && rails g model Provider
```
be sure to migrate the db afterwards

```
$ bundle exec rake db:migrate
```

We have one side of our polymorphic relationship, now we need to update
the other side of it with our new User types.


```ruby app/models/customer.rb
class Customer < ActiveRecord::Base
  has_one :user, as: :profile, dependent: destroy
  accepts_nested_attributes_for :user
end
```

```ruby app/models/provider.rb
class Provider < ActiveRecord::Base
  has_one :user, as: :profile, dependnet: destroy
  accepts_nested_attributes_for :user
end
```

The first line is filling out the other half of our relationship to the
users.  We're telling Rails it has one `User` and `as: profile`
indicates it's a polymorphic relationship.  We've also added `dependent:
destroy` to make sure when we delete an object, and associated objects
are deleted as well.

The second line we added is needed to create an associated `User`
whenever we create either a `Customer` or `Provider`.

`accepts_nested_attributes_for` allow you to save attributes on associated records through the parent.
[accepts_nested_attributes_for](http://api.rubyonrails.org/classes/ActiveRecord/NestedAttributes/ClassMethods.html)

Back to an earlier topic of strong parameters. We now need to
whitelist the attributes for the `User` inside the each of our
controllers.

We already have our models created, but we would still like to generate a
lot of the boilerplate we need to create the controller and views so we
can keep plugging away at our project.  Using the `rails scaffold`
command, would create a model for whatever resource name we specify, so
what are our alternatives.  Let's ask for some help...

//why is this still showing the other filename?

```
$ rails generate -h
```

This gives us a lot of options for the generate command. Take a look this
section in particular:

```
Please choose a generator below.

Rails:
  assets
  controller
  generator
  helper
  integration_test
  job
  mailer
  migration
  model
  resource
  responders_controller
  scaffold
  scaffold_controller
  task
```
  
Take a look at `scaffold_controller`,  here's what it does:

>Stubs out a scaffolded controller and its views. Pass the model name, either CamelCased or under_scored, and a list of views as arguments. The controller name is retrieved as a pluralized version of the model name.  To create a controller within a module specify the model name as a path like 'parent_module/controller_name'.  This generates a controller class in app/controllers and invokes helper, template engine and test framework generators.

Running:
```
$ rails g scaffold_controller Customer
```
and

```
$ rails g scaffold_controller Provider
```

will scaffold a controller with all our RESTful routes along with
their associated views.  If you look at the created files, there weren't
and migrations or changes to the model, just our controllers and views.

However, we do have to specify our routes, so let's add those in now:

```ruby config/routes.rb
  resources :customers
  resources :providers
```

With our routes set up, let's update both of our forms to collect the
sign up info from our user.


```html app/views/customers/_form.html.erb
<%= form_for(@customer) do |f| %>
  <% if @customer.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(@customer.errors.count, "error") %> prohibited this customer from being saved:</h2>

      <ul>
        <% @customer.errors.full_messages.each do |message| %>
          <li><%= message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <div class="panel-body" style="text-align: center;">
    <%= f.fields_for :user do |u| %>
      <div class="form-group">
        <%= u.label :first_name %><br>
        <%= u.text_field :first_name, class: 'form-control' %>
      </div>

      <div class="form-group">
        <%= u.label :last_name %><br>
        <%= u.text_field :last_name, class: 'form-control' %>
      </div>

      <div class="form-group">
        <%= u.label :email %><br>
        <%= u.email_field :email, class: 'form-control' %>
      </div>

      <div class="form-group">
        <%= u.label :password %><br>
        <%= u.password_field :password, class: 'form-control' %>
      </div>
    <% end %>

    <div class="form-group" style="margin: 10px 0px; text-align: center;">
      <%= f.submit "Submit", :class=>"btn btn-info" %>
    </div>

    <% if request.path == new_customer_path %>
      <%= link_to "Log in", new_user_session_path %><br />
    <% end %>
  </div>

<% end %>

```

This may look a little odd because asll the elements listed in the form
are nested.  This is becasue right now, we dont have any fields specific
to the Customer.  When you add your own elements to the
Customer/Provider, it will look something like this.

```html
<%= f.label :home_address %>
<%= f.text_field :home_address %>

  <%= f.fields_for :user do |u| %>
    ...
  <% end %>

  ...
<% end %>
```
Go ahead and do the same thing with the `Provider` form as well.  Once that's
complete, we're ready to start doing work on our forms now.

If you look at your forms on the front end.  Unless you added attributes
for either the Provider or Customer, you wont see any fields.  This is
because right now, we havn't done anyhting to build a `User` to go along
with the association we're trying to create.

Build the associated user inside you `#new` method so it's available to
the form.


```ruby app/controllers/customers_controller.rb
  def new
    @customer = Customer.new
    @customer.build_user
  end
```

and to allow user updates to be allowed through our `Customer` model, we
need to add those to our strong params.

```ruby app/controllers/customers_controller.rb
    def customer_params
      params.require(:customer).permit(user_attributes: [:first_name,
                                                         :last_name,
                                                         :email,
                                                         :password])
    end
```
If you go to the `new_customer_path`, we can create a new `Customer`
and `User` at the same time, and get redirected to the `show` view for the
customer we just created.  If you look in the top corner of the nav bar,
we still see the Sign In link, whereas, if we were logged in, we would
see our email address.  We need to make one more update to
create a new `Customer`, create an associated `User`, and create a new
session for the `User` we just created.  This sounds a lot more
complicated than it actually is.  Devise give us a very handy `sign_in`
method.

```ruby
 # POST /customers
  def create
    @customer = Customer.new(customer_params)
    if @customer.save
      flash[:success] = 'Customer was successfully created.'
      sign_in(@customer.user)
      redirect_to @customer
    else
      render :new
    end
  end

```

and that's that!  Now, when you go to create a new `Customer` or
`Provider`, we automatically create an associated `User` and sign in.
One of my favorite things about this set up is you, can use single form
to sign in at the `new_user_session` path.

Again, this is by no means an exhaustive post on Polymorphism or Devise.
Just my way of combining two common uses into a practical example
There are a ton of great resources online if you'd like to do a deeper
dive into either topics.
