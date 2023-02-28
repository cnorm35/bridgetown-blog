---
layout: post
title:  Adding dark mode to your Rails application with Tailwind and Stimulus
date:   2023-02-27 16:54:46 -0500
category: ruby
excerpt: "This post walks through the steps to add a dark-mode toggle to your
Rails app with Tailwind and Stimulus JS.  On top of actually implementing the
ability to toggle back and forth, I'll also go over some ways you can create
your own custom dark theme"
author: cody
---
Psssst....

Hey you.....

*opens jacket*.....want some dark-mode?

If you _really_ want some, you can skip the article and view the example repo
[here](https://github.com/cnorm35/tailwind_darkmode_example)

These days, it seems like every site on the internet has a toggle for switching
to dark-mode.  There are a lot of reasons for it.  It can conserve power, lower
eye strain, and let's face it, looks cool.

Are you looking for some sweet dark-mode action with Tailwind and Stimulus JS but not sure where to start?
Well, you've come to the right place.

### Inspiration
Before we get started, I'd like to credit this [article](https://www.freecodecamp.org/news/how-to-build-a-dark-mode-switcher-with-tailwind-css-and-flowbite/)
for the inspiration for this post. When I was looking for some resources on how
to add dark-mode for my own app, it was a big help so thank you Zoltán!

With that post and some information on dark-mode straight from [Tailwind](https://tailwindcss.com/docs/dark-mode#toggling-dark-mode-manually),
we have the info we need to start embracing the dark side.

To start, let's create a fresh rails app with Tailwind and get started.

### Getting Started

In your terminal, run the following command:


`$ bin/rails new tailwind_darkmode --css tailwind`

This creates a fresh Rails app with Tailwind already installed and ready to go.  At the time of writing this, it will be a Rails 7 app with importmaps and SQLite DB.

Before we can start adding some dark styles, we have to add some views to add styles to.

Create a Static controller with an index page. This will be the root path and the only
view we'll focus on.

`$ bin/rails g controller Static index`

After the generator completes, open the routes file in` config/routes.rb` and make our new view we just created the root path.

`config/routes.rb`
```ruby
Rails.application.routes.draw do
  root "static#index"
end
```

Keep in mind, When adding changes to your routes file, be sure to restat your server for your changes to take effect.

### Adding your first Tailwind view

Now our new page is the root path by default, let's get to adding some Tailwind
styles. In `app/views/static/index.html.erb` replace the code generated when we
created the view and add the following code for the hero section.


`app/views/static/index.html.erb`
```html
<div class="bg-gray-100">
  <div class="container mx-auto flex flex-col items-center py-12 sm:py-24">
    <div class="w-11/12 sm:w-2/3 lg:flex justify-center items-center flex-col  mb-5 sm:mb-10">
      <h1 class="text-2xl sm:text-3xl md:text-4xl lg:text-5xl xl:text-6xl text-center text-gray-800 font-black leading-7 md:leading-10">
          Adding
          <span class="text-purple-600">Dark</span>
          Mode to your Rails apps with Tailwind
      </h1>
      <p class="mt-5 sm:mt-10 lg:w-10/12 text-gray-400 font-normal text-center text-sm sm:text-lg">Dark Mode not only looks cool, but can help reduce eye strain and conserve battery life.</p>
    </div>
    <div class="flex justify-center items-center">
      <button class="focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-purple-700 bg-purple-700 transition duration-150 ease-in-out hover:bg-purple-600 lg:text-xl lg:font-bold  rounded text-white px-4 sm:px-10 border border-purple-700 py-2 sm:py-4 text-sm">Get Started</button>
      <button class="ml-4 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-purple-700 bg-transparent transition duration-150 ease-in-out hover:border-purple-600 lg:text-xl lg:font-bold  hover:text-purple-600 rounded border border-purple-700 text-purple-700 px-4 sm:px-10 py-2 sm:py-4 text-sm">Live Demo</button>
    </div>
  </div>
</div>

```

<img
  alt="Tailwind Hero Section"
  class="position-relative mx-auto rounded w-100 shadow-lg"
  src="https://personal-blog-assets.s3.amazonaws.com/DarkModeLightBackground.png"
/>


Here is one of the many cool things about tailwind.  Tailwind includes a `dark`
variant that allows you specify the styles when dark-mode is enabled.

By default this uses the `prefer-color-scheme` [CSS media feature](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-color-scheme).

This will automatically detect if you have your OS or browser settings
enabled for dark-mode and use that automatically.


### Adding styles for dark-mode

If you don't care about the option to toggle back and forth and just display a
dark version of the site if the user has that set somewhere else, all that's
left is to add your styling for dark-mode. To see if everything is configured
and working correctly, you can add a dark-mode style to the body and see if it
updates. Add a dark style to the body in `app/views/layouts/application.html.erb`

```html
  <body class="dark:bg-black">
    <main class="container mx-auto mt-28 px-5 flex">
      <%= yield %>
    </main>
  </body>
```

If you have dark-mode enabled for your browser or OS, the page should look
something like this:


If you aren't looking to add the ability to toggle back and forth, all that's
left is to start adding dark styles to your app.  It's not _quite_ as simple as
just inverting the colors, there's a surprising amount of fine-tuning that goes
into a nice dark theme. I'm by no means an expert but have picked up a couple
of tricks to help you figure out what looks good.

Defaulting to the user's OS or browser preference is nice but variety is the
spice of life.  With a few extra steps, we can add the ability to toggle back
and forth depending on our fancy.

### Adding option to toggle between Light and Dark

Tailwind needs a couple of changes to know that dark-mode styles are used with
classes instead of the user's preferences and media queries.

https://tailwindcss.com/docs/dark-mode#toggling-dark-mode-manually

To allow us to toggle back and forth, we'll need to update some Tailwind
settings.  Add `darkMode: 'class'` to `tailwind.config.js`

Here is what my `config/tailwind.config.js` file looks like

```js
const defaultTheme = require('tailwindcss/defaultTheme')

module.exports = {
  darkMode: 'class',
  content: [
    './public/*.html',
    './app/helpers/**/*.rb',
    './app/javascript/**/*.js',
    './app/views/**/*.{erb,haml,html,slim}'
  ],
  theme: {
    extend: {
      fontFamily: {
        sans: ['Inter var', ...defaultTheme.fontFamily.sans],
      },
    },
  },
  plugins: [
    require('@tailwindcss/forms'),
    require('@tailwindcss/aspect-ratio'),
    require('@tailwindcss/typography'),
  ]
}
```

You might notice that once you've updated this setting, Tailwind no longer
defaults to the `prefers-color-selction`. If you refresh and visit the homepage,
even if you have your browser or OS settings preferring dark styles, the dark
styles are not applied automatically.  This is what we want.  That means
Tailwind is now looking for a `dark` class which is what we will use to toggle
back and forth.

When toggling between dark and light styles, Tailwind looks for the `dark` class
on a parent element.  In other words, somewhere, _above_ where the dark styles
are to be applied.

I like the option of having a mix of these two approaches.  Defaulting to the
user's system preference is present and still offering a manual selection.
Lukck for me, so do the fine folks at Tailwind

Tailwind has a great JS snippet for just that.

```js
// On page load or when changing themes, best to add inline in `head` to avoid FOUC
if (localStorage.theme === 'dark' || (!('theme' in localStorage) && window.matchMedia('(prefers-color-scheme: dark)').matches)) {
  document.documentElement.classList.add('dark')
} else {
  document.documentElement.classList.remove('dark')
}

// Whenever the user explicitly chooses light mode
localStorage.theme = 'light'

// Whenever the user explicitly chooses dark mode
localStorage.theme = 'dark'

// Whenever the user explicitly chooses to respect the OS preference
localStorage.removeItem('theme')

```

If you read over the comments in the snippet above, you'll see a note about
adding this to the head of our HTML to prevent FOUC or Flashes Of Unstyled Content.

You can insert that snippet directly into the head of the HTML in your layout,
or you can add it to a partial that's rendered within the head.  This is my
preferred approach to keep things in my layout clean but it's more of a
preference than a requirement.

```html
  <head>
    <title>TailwindDarkmode</title>
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>
    <%= stylesheet_link_tag "tailwind", "inter-font", "data-turbo-track": "reload" %>

    <%= stylesheet_link_tag "application", "data-turbo-track": "reload" %>
    <%= javascript_importmap_tags %>
    <%= render "shared/dark_mode_check" %>
  </head>
```

After adding this code to the header, if you have your OS or browser settings to
prefer dark colors, you'll notice that the app is back to defaulting to that
choice.  Now we have the option to toggle back and forth and check the user's
preferences.

Next up in our journey to the dark side is to build something to toggle back and
forth.

### Building the switch

Up until now, most of this info has been right from the tailwind docs.  A lot of
this was lifted from the other blog post so thanks for that homie! Cite and
share your work!

Before jumping in, let's break down what this toggle feature will consist of.

- Clicking some element toggles from light to dark
- Swap The Icon
- Add or remove the `dark` class to our body tag.

Since there is already a `shared` directory, that seems like a nice place to put
this partial.  The new file will be called `app/views/shared/_darkmode_toggle.html.erb`

This view was pulled from the Flowbite tutorial above

`app/views/shared/_darkmode_toggle.html.erb`

```html
<button
  id="app-darkmode-toggle"
  type="button"
  class="align-middle text-gray-300 dark:text-gray-300 hover:bg-gray-100 dark:hover:bg-gray-700 focus:outline-none focus:ring-4 focus:ring-gray-200 dark:focus:ring-gray-700 rounded-lg text-sm p-2.5"
  data-dark-mode-target="themeToggle"
  data-action="click->dark-mode#toggleTheme"
>
  <svg
    class="w-5 h-5 hiden"
    fill="currentColor"
    viewBox="0 0 20 20"
    xmlns="http://www.w3.org/2000/svg"
    data-dark-mode-target="darkIcon"
  >
    <path
      d="M17.293 13.293A8 8 0 016.707 2.707a8.001 8.001 0 1010.586 10.586z"
    ></path>
  </svg>
  <svg
    class="w-5 h-5 hidden"
    fill="currentColor"
    viewBox="0 0 20 20"
    xmlns="http://www.w3.org/2000/svg"
    data-dark-mode-target="lightIcon"
  >
    <path
      d="M10 2a1 1 0 011 1v1a1 1 0 11-2 0V3a1 1 0 011-1zm4 8a4 4 0 11-8 0 4 4 0 018 0zm-.464 4.95l.707.707a1 1 0 001.414-1.414l-.707-.707a1 1 0 00-1.414 1.414zm2.12-10.607a1 1 0 010 1.414l-.706.707a1 1 0 11-1.414-1.414l.707-.707a1 1 0 011.414 0zM17 11a1 1 0 100-2h-1a1 1 0 100 2h1zm-7 4a1 1 0 011 1v1a1 1 0 11-2 0v-1a1 1 0 011-1zM5.05 6.464A1 1 0 106.465 5.05l-.708-.707a1 1 0 00-1.414 1.414l.707.707zm1.414 8.486l-.707.707a1 1 0 01-1.414-1.414l.707-.707a1 1 0 011.414 1.414zM4 11a1 1 0 100-2H3a1 1 0 000 2h1z"
      fill-rule="evenodd"
      clip-rule="evenodd"
    ></path>
  </svg>
</button>
```

This includes some of the data attributes we'll use with Stimulus later on.

Now that there's a partial that has both of the buttons to toggle back and
forth, I like to add it to our view  and make sure everything is displaying
correctly.

With the code as it is right now, both of the icons have the `hidden` class so
they won't be visible.  Removing the `hidden` class from both of those buttons
allows you to check the placement and make sure the SVGs for the icons are
rendering correctly.

<img
  alt="Tailwind Dark Hero Section"
  class="position-relative mx-auto rounded w-100 shadow-lg"
  src="https://personal-blog-assets.s3.amazonaws.com/DarkModeDarkBackground.png"
/>

### Toggle with Stimulus JS

Now comes the JS.  To get started with the JS for toggling back and forth
between dark and light mode, generate a new Stimulus controller with the
following command:

`bin/rails generate stimulus dark_mode`

This creates a new Stimulus controller at
`app/javascript/controllers/dark_mode_controller.js`

```js
import { Controller } from "@hotwired/stimulus"

// Connects to data-controller="dark-mode"
export default class extends Controller {
  connect() {
  }
}
```

Now, let's adapt the code from the other blog post to work in the Stimulus
controller.

Update the `dark_mode_controller.js` file to look like this:


```js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = [ "lightIcon", "darkIcon", "themeToggle" ]

  connect() {
    if (localStorage.getItem('color-theme') === 'dark' || (!('color-theme' in localStorage) && window.matchMedia('(prefers-color-scheme: dark)').matches)) {
      this.lightIconTarget.classList.remove('hidden');
    } else {
      this.darkIconTarget.classList.remove('hidden');
    }
  }

  toggleTheme() {
    console.log('theme target clicked')
    this.lightIconTarget.classList.toggle('hidden');
    this.darkIconTarget.classList.toggle('hidden');
    if (localStorage.getItem('color-theme')) {
        if (localStorage.getItem('color-theme') === 'light') {
            document.documentElement.classList.add('dark');
            localStorage.setItem('color-theme', 'dark');
        } else {
            document.documentElement.classList.remove('dark');
            localStorage.setItem('color-theme', 'light');
        }

    // if NOT set via local storage previously
    } else {
        if (document.documentElement.classList.contains('dark')) {
            document.documentElement.classList.remove('dark');
            localStorage.setItem('color-theme', 'light');
        } else {
            document.documentElement.classList.add('dark');
            localStorage.setItem('color-theme', 'dark');
        }
    }
  }
}
```

Before attaching the Stimulus controller to an element on the page, let's break
down what's happening.

```js
  static targets = [ "lightIcon", "darkIcon", "themeToggle" ]
```

These are our Stimulus
[targets](https://stimulus.hotwired.dev/reference/targets).  These are the
elements on the page that we'll be interating with by adding or removing
classes.

```js
  connect() {
    if (localStorage.getItem('color-theme') === 'dark' || (!('color-theme' in localStorage) && window.matchMedia('(prefers-color-scheme: dark)').matches)) {
      this.lightIconTarget.classList.remove('hidden');
    } else {
      this.darkIconTarget.classList.remove('hidden');
    }
  }
```

This is the `connect` [method](https://stimulus.hotwired.dev/reference/lifecycle-callbacks) of the Stimulus controller.  This code will be executed anytime the controller connects to the DOM.

The first line looks for dark theme preferences saved to local storage (like in
our script we added to the header) or `prefers-color-scheme` in our browser
settings. If so, removes the hidden class to the light icon.  This is how we
reference the `lightIcon` target from above.  The logic is also a little
confusing without seeing it in action.

If nothing is set, we remove the `hidden` class for the `darkIcon`.

If you currently have dark theme preferences set, _display the light icon_ to
toggle to the light theme, instead of displaying the current theme.  It makes
sense seeing it in action, but can look a little odd at first glance so wanted
to point that out.

The next method is the main focus of the controller, the `toggleTheme()` method

```js
toggleTheme() {
    console.log('theme target clicked')
    this.lightIconTarget.classList.toggle('hidden');
    this.darkIconTarget.classList.toggle('hidden');
    if (localStorage.getItem('color-theme')) {
        if (localStorage.getItem('color-theme') === 'light') {
            document.documentElement.classList.add('dark');
            localStorage.setItem('color-theme', 'dark');
        } else {
            document.documentElement.classList.remove('dark');
            localStorage.setItem('color-theme', 'light');
        }

    // if NOT set via local storage previously
    } else {
        if (document.documentElement.classList.contains('dark')) {
            document.documentElement.classList.remove('dark');
            localStorage.setItem('color-theme', 'light');
        } else {
            document.documentElement.classList.add('dark');
            localStorage.setItem('color-theme', 'dark');
        }
    }
  }
```


The first thing that happens in this controller is logging to the console that
the method was called.  This is helpful for ensuring that your action ran
at the correct time, like clicking the theme toggle button.

The next part is to add the `hidden` class to `both` the light and dark-mode.
Hiding both allows us to make some checks to determine if the `lightIcon` or
`darkIcon` should be displayed.  Again, the browser localStorage is checked for
an existing `color-theme` with `localStorage.getItem('color-theme')`.  If one if
found, the next step is to see if the value is `'light'` or `'dark'`.

Depending on that value, the next step either adds or removes the `dark` class
and sets the theme to the `localStorage` in the browser.

Now we have the JS in the stimulus controller, we need to add the correct data
attributes to the HTML element, then tie a click event to it.

With a better idea of what's happening inside the Stimulus controller and how
Tailwind toggles back and forth, it's time to attach this Stimulus controller to
our dark-mode toggle buttons so we can see this in action.

To attach the Stimulus controller to the `body` tag, we add the data attribute
mentioned in the comment after we first generated the Stimulus controller
`data-controller="dark-mode"`


`app/views/layouts/application.html.erb`
```html
  <body class="dark:bg-black" data-controller="dark-mode">
    <main class="flex-1">
      <div class="max-w-7xl mx-auto px-4 sm:px-6 md:px-8">
        <%= render "shared/darkmode_toggle" %>
      </div>
      <%= yield %>
    </main>
  </body>
```

This just connects the Stimulus controller to the body tag in our view, we still
need to add the action to toggle back and forth with `toggleTheme`

For that, we need to add a click event to the toggle button we added earlier.

If you copied the markup from above for the dark-mode toggle, then all the data
attributes you need are already in place.  If those are in place, things should
be toggling back and forth now.  Here is what those data attributes are doing.

`data-dark-mode-target="themeToggle"`

This tells Stimulus that this is the `themeToggle` target in the controller
`dark-mode`


`data-action="click->dark-mode#toggleTheme"` is where we add a click event to
the button.  When this button is clicked, the `toggleTheme` within the
`dark-mode` Stimulus controller will be run.


Clicking on the button should toggle the theme between dark and light mode now.
If you've been following along, clicking the toggle should toggle between a
black background and a white one. And now you have a fancy new dark-mode...sort
of.


### HTML vs ERB views
Before moving on to how I start refining the look of my dark-mode theme, I'd
like to point out one thing about the views added so far.

You can, and probably should have more of these views with more erb than HTML
(think buttons and divs).

I used to write a lot more of my views with erb, but being able to copy TailwindUI HTML right into my views with
minimal tweaking to get it to look decent, has been faster and easier way
for me to keep things rolling.  It's probably not the best, but for me, was an
acceptable tradeoff for the speed of adding new changes it gives me so for me,
that makes it worth it.

### Refining your colors

I mentioned it briefly before, but having a nice dark theme for your app isn't
quite as easy as just making backgrounds black with text white. For the older
folks reading this, it gives your site a real Geocities vibe.  You might as well
throw in a site counter.

There's a surprising amount of finesse that goes into getting a dark theme to look
right and I have a ton of respect for those that can do it well.  I don't
consider myself part of that group.  Maybe you don't either and you need some
help figuring out how you would like your site to look.

Tailwind ships out of the box with a top-notch [color pallatte](https://tailwindcss.com/docs/customizing-colors) and we also have
the ability to add some custom colors, once we figure out what we'd like those
custom colors to be.

You can probably pick a couple of those and start making some big improvements.
That's a good option, but there's another way I like to do it by adding some custom
colors.

I've been using a couple of different dark-mode plugins for years.  It seems vastly reduce my
eye strain so I've been using it for years now.

As long as I can remember, I've usually preferred dark themes, they just seem to
be much easier on my eyes. It seems over the past couple of years, a lot more
sites are supporting dark-mode options, but that didn't use to be the case so I
usually use a browser plugin to convert all sites to dark-mode. The main one
I've been using is [Dark Reader](https://darkreader.org/)

I think it does a great job at converting sites to dark-mode, so much so that I
decided to take a little inspiration from it.

For a good starting point for creating a nice dark mode theme, I usually open the
site with the Dark Reader active, open my inspector, and use a color-dropper to
get the color codes the Dark Reader plugin is using.

I'll make note of those hex codes and add them as [custom
colors](https://tailwindcss.com/docs/customizing-colors) in Tailwind

Another good resource for color inspiration is this handy Tailwind [color shade
generator](https://javisperez.github.io/tailwindcolorshades)

After adding more of your dark styles, you'll probably start to notice a few
spots where different states don't have any dark syles,  think `hover`, `focus`, `active` and
others.

For these cases, you add the state your targeting after `dark`

`dark:hover:bg-gray-600` or `dark:focus:ring-blue-500`

### Final Thoughts

Once you have an idea of the colors you'd like to use for your dark mode
variant, the easiest way is to add the dark variants into `@apply` and make a
class.  That's a good option once you know what you want but until then, you'll
probably spend a lot of time flipping back and forth between light and dark mode
filling in all the spots you missed.  That's normal and part of the process that
keeps it looking nicer than something that's just like a dark background with
white text.

Rails , Hotwire and Tailwind makes me feel like I have superpowers these
days. I think this is a great feature and there are a lot of benefits to dark-mode like battery life, eye strain and badassness (debatable).  I hope this
helps and works well for you.
