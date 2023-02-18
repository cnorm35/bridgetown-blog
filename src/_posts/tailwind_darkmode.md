---
layout: post
title:  Adding darkmode to tailwind rails stimulus
date:   2023-02-10 16:54:46 -0500
category: ruby
excerpt: "Add dark mode for tailwind to your rails app"
author: cody
---

These days, it seems like every site on the internet has a toggle for switching
to dark mode.  There are a lot of reasons for it.  It can conserve power, lower
eye strain and lets' face it, looks cool as shit.

Want some sweet dark mode action with Tailwind but not sure where to start?
You've come to the right place.

Before we get started, I'd like to credit this article
https://www.freecodecamp.org/news/how-to-build-a-dark-mode-switcher-with-tailwind-css-and-flowbite/
for the inspiratin for this post.

With that post and some information on dark mode straight from Tailwind https://tailwindcss.com/docs/dark-mode#toggling-dark-mode-manually

We just (<- fix that, it could be really hard) need to convert that to stimulus
JS and add some styles for dark mode.

<!-- (Use regular jumpstart as a template) https://github.com/excid3/jumpstart -->

`rails new tailwind_darkmode --css tailwind --database=postgresql`


To get started, we'll create a new rails app from the dope jumpstart template
with postgres and tailwind.  At the time of writing this, my rails version is
7.0

Now that everything is enabled, let's make a couple of changs to see if
everything is working as it should.

Actually think a lot of things are a little funky with the bootstrap stuff being
included in.  maybe just do something like fresh rails app install instead?

(Talk about updating for darkmode class)

Create a controller for Static with an index page, that's the root and they only
view we'll focus on.

Something like `$ be rails g controller Static index`

This will create a Static controller with an index view.  This is where the hero
will go.

Add a hero section to your index page to get some tailwind styles going

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
      <p class="mt-5 sm:mt-10 lg:w-10/12 text-gray-400 font-normal text-center text-sm sm:text-lg">Dark Mode not only looks cool, but can help reduce eye strain.</p>
    </div>
    <div class="flex justify-center items-center">
      <button class="focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-purple-700 bg-purple-700 transition duration-150 ease-in-out hover:bg-purple-600 lg:text-xl lg:font-bold  rounded text-white px-4 sm:px-10 border border-purple-700 py-2 sm:py-4 text-sm">Get Started</button>
      <button class="ml-4 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-purple-700 bg-transparent transition duration-150 ease-in-out hover:border-purple-600 lg:text-xl lg:font-bold  hover:text-purple-600 rounded border border-purple-700 text-purple-700 px-4 sm:px-10 py-2 sm:py-4 text-sm">Live Demo</button>
    </div>
  </div>
</div>

```

^ Add a preview image for what it should look like afterwards

After adding your view, updte your routes to root to this new index page to
keep things simple.

`config/routes.rb`
```
root "static#index"
```

Now here is one of the cool things about tailwind.  Tailwind includes a `dark`
variant that let's you specify the styles when dark mode is enabled.  By
default this uses the `prefer-color-scheme` CSS media feature. https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-color-scheme

This will automatically detect if you have your operationg or browser settings
enabled for dark mode and use that automatically.

If you don't care about the option to toggle back and forth and just display a
dark version of the site if the user has that set somewhere else, all that's
left is to add your styling for dark mode. To see if everything is configured
and working correctly, you can add a dark mode style to the body and see if it
updates

```html
  <body class="dark:bg-black">
    <main class="container mx-auto mt-28 px-5 flex">
      <%= yield %>
    </main>
  </body>
```

If you have dark mode enabled for your browser or OS, the page should look
something like this:

IMAGE HERE

I'll talk more about _how_ to style for dark mode, or at least how I did it so
if you want to skip over toggling between dark and light click HERE.  It's not
quite as straightforward as making things that are white black and vice versa,
at least not if you want things to look good.


Now that we have the power of the dark side, let's (i don't know if i like how i
keep useing lets) work on adding the option to switch back and forth.


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

You might notice that once you've updated this setting, Tailwind will no longer
default to the prefers-color-selction (or whatever) so the background color for
the body should no longer be black.  There should not be any dark styles active
even if you have your settings for it,  we'll need to toggle that on and off
now.

The thing that does da togglin' is looking for the `dark` class in one of the
parent elements.  In other words, somewhere _above_ where the dark styling is
present.

I like the option of having a mix of these two approaches.  Defaulting to the
user's system preference is present and still offering a manual selection.

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

As is says in the comment, to avoid some issues, we're going to add this as a
script tag the head of our HTML


```html
  <head>
    <title>TailwindDarkmode</title>
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>
    <%= stylesheet_link_tag "tailwind", "inter-font", "data-turbo-track": "reload" %>

    <%= stylesheet_link_tag "application", "data-turbo-track": "reload" %>
    <%= javascript_importmap_tags %>
    <script>

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
    </script>
  </head>
```

In normal situations, I would add that in a partial and render the partial to
keep things clean but since the focus today is on dark mode, we're just gonna
send it and keep rolling on.

After adding that snippet, our app will look for a dark mode system setting and
if present use it again.  We have the beginning of the option to toggle back and
forth


### Building the switch

Up until now, most of this info has been right from the tailwind docs.  A lot of
this was lifted from the other blog post so thanks for that homie! Cite and
share your work!

(Pick back up with adding button and toggle)


(Talk about what dark mode toggle is at its basics 

Clicking an element 
Swapping the icon to show the current state 
Add or remove the dark class )

Let's break down what this toggle feature will consist of.

- Clicking some element toggles from light to dark
- Swap The Icon to the current theme
- Add or remove the `dark` class to our body tag.


To start with, let's (I really hate that <-) start with creating a partial for
our dark mode toggle

For something like this I'll usually put it in a `shared` folder so the new file
would be `app/views/shared/_darkmode_toggle.html.erb`


```html
<button
  id="app-darkmode-toggle"
  type="button"
  class="align-middle text-gray-300 dark:text-gray-300 hover:bg-gray-100 dark:hover:bg-gray-700 focus:outline-none focus:ring-4 focus:ring-gray-200 dark:focus:ring-gray-700 rounded-lg text-sm p-2.5"
  data-dark-modetarget="themeToggle"
  data-action="click->darkmode#toggleTheme"
>
  <svg
    class="w-5 h-5 hidden"
    fill="currentColor"
    viewBox="0 0 20 20"
    xmlns="http://www.w3.org/2000/svg"
    data-darkmode-target="darkIcon"
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
    data-darkmode-target="lightIcon"
  >
    <path
      d="M10 2a1 1 0 011 1v1a1 1 0 11-2 0V3a1 1 0 011-1zm4 8a4 4 0 11-8 0 4 4 0 018 0zm-.464 4.95l.707.707a1 1 0 001.414-1.414l-.707-.707a1 1 0 00-1.414 1.414zm2.12-10.607a1 1 0 010 1.414l-.706.707a1 1 0 11-1.414-1.414l.707-.707a1 1 0 011.414 0zM17 11a1 1 0 100-2h-1a1 1 0 100 2h1zm-7 4a1 1 0 011 1v1a1 1 0 11-2 0v-1a1 1 0 011-1zM5.05 6.464A1 1 0 106.465 5.05l-.708-.707a1 1 0 00-1.414 1.414l.707.707zm1.414 8.486l-.707.707a1 1 0 01-1.414-1.414l.707-.707a1 1 0 011.414 1.414zM4 11a1 1 0 100-2H3a1 1 0 000 2h1z"
      fill-rule="evenodd"
      clip-rule="evenodd"
    ></path>
  </svg>
</button>
```


With the code as it is right now, both of the icons have the `hidden` class so
they won't be visible.  If you'd like to check the placement of the icons, you
can either remove the `hidden` class or finish the implementation that adds or
removes that class.



Maybe just make the button the 'Get Started' one?



`bin/rails generate stimulus darkmode`

Now, we'll take the JS from the article above and adapt it to stimulus.



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

Now we have the JS in the stimulus controller, we need to add the correct data
attributes to the HTML element, then tie a click event to it.


First we attcah the stimulus controller to the body so we can add or remove
the dark class.  Also add the partial for the toggle in there too

`app/views/layouts/application.html.erb`
```html
  <body class="dark:bg-black" data-controller="darkmode">
    <main class="flex-1">
      <div class="max-w-7xl mx-auto px-4 sm:px-6 md:px-8">
        <%= render "shared/darkmode_toggle" %>
      </div>
      <%= yield %>
    </main>
  </body>
```


<!-- Something like this would work without the icon -->
<!-- ``` -->
<!-- import { Controller } from "@hotwired/stimulus" -->

<!-- export default class extends Controller { -->
<!--   // static targets = [ "lightIcon", "darkIcon", "themeToggle" ] -->
<!--   static targets = [ "themeToggle" ] -->

<!--   connect() { -->
<!--     // // this.lightIconTarget.classList.toggle('hidden') -->
<!--     // this.darkIconTarget.classList.toggle('hidden') -->
<!--     // Change the icons inside the button based on previous settings -->
<!--     if (localStorage.getItem('color-theme') === 'dark' || (!('color-theme' in localStorage) && window.matchMedia('(prefers-color-scheme: dark)').matches)) { -->
<!--       // this.lightIconTarget.classList.remove('hidden'); -->
<!--     } else { -->
<!--       // this.darkIconTarget.classList.remove('hidden'); -->
<!--     } -->
<!--   } -->

<!--   toggleTheme() { -->
<!--     console.log('theme target clicked') -->
<!--     // this.lightIconTarget.classList.toggle('hidden'); -->
<!--     // this.darkIconTarget.classList.toggle('hidden'); -->
<!--     // maybe move? -->
<!--     if (localStorage.getItem('color-theme')) { -->
<!--         if (localStorage.getItem('color-theme') === 'light') { -->
<!--             document.documentElement.classList.add('dark'); -->
<!--             localStorage.setItem('color-theme', 'dark'); -->
<!--         } else { -->
<!--             document.documentElement.classList.remove('dark'); -->
<!--             localStorage.setItem('color-theme', 'light'); -->
<!--         } -->

<!--     // if NOT set via local storage previously -->
<!--     } else { -->
<!--         if (document.documentElement.classList.contains('dark')) { -->
<!--             document.documentElement.classList.remove('dark'); -->
<!--             localStorage.setItem('color-theme', 'light'); -->
<!--         } else { -->
<!--             document.documentElement.classList.add('dark'); -->
<!--             localStorage.setItem('color-theme', 'dark'); -->
<!--         } -->
<!--     } -->
<!--   } -->
<!-- } -->

<!-- ``` -->

<!-- Add that to the JS controller. -->

Add the click event to the button 
```html
      <button class="focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-purple-700 bg-purple-700 transition duration-150 ease-in-out hover:bg-purple-600 lg:text-xl lg:font-bold  rounded text-white px-4 sm:px-10 border border-purple-700 py-2 sm:py-4 text-sm" data-action="click->darkmode#toggleTheme" data-darkmode-target="themeToggle">Get Started</button>

```


Here is something that might work for the layout part (the body snippet above
should work)

<!-- ```html -->
<!--   <body class="h-full font-sans antialiased font-normal leading-normal bg-gray-50 dark:bg-landing-page-dark" data-controller="dark-mode"> -->
<!--     <div class="main flex flex-col"> -->
<!--       <%= render partial: "shared/flash" %> -->

<!--         <main class="flex-1"> -->
<!--           <div class="py-6"> -->
<!--             <div class="max-w-7xl mx-auto px-4 sm:px-6 md:px-8"> -->
<!--               <%= render "shared/app_dark_mode_toggle" %> -->

<!-- ``` -->

Both of the icons start out hidden.  When our stimulus component connects, we
check for users settings for dark mode and if we find one, show things with the
dark theme.  When dark mode is active, we show the Sun or light icon.  Clicking
on that converts things to light mode.



Update the views with some dark styles.

Clicking on the button should toggle, that's fine. If you want you can finish
here or you can stick around to see how to update and show if it's a light or
dark theme and toggle back and forth.

After that, I'm going to go into _how_ I choose the colors for a dark mode.
While you could defintely just make everything have a black background with
whitetext, but it's going to look closer to a geocities site than a slick modern
front end.

We can do that with a combination of the stock tailwind colors if we have the
plugin added or we can add some of our custom colors.


You can, and probably should have more of these views in erb. I used to do that
a lot more but being able to copy TailwindUI html right into my views with
minimal tweaking to get to look decent, that's been a much faster and easier way
for me to keep things rolling.  It's probably not the best, but it's not the
worst and it's what I've been doing.



Now the power of dark mode toggling is yours.  Mentioned above, it's not always
as straightforward as just making backgrounds black with white text.

Tailwind ships with a great color pallatte https://tailwindcss.com/docs/customizing-colors

You need to require the colors from tailwind:
`const color = require("tailwindcss/colors")`

You can probably pick a couple of those and start making some big improvements.
That's a good option, but there's another way I like to do it adding some custom
colors.

Before every site started adding the option of darkmode, I've been using a
couple of different dark mode plugins for years.  It seems to really help my
eye strain so I've been using it for years now.

I really like the Dark Reader one https://darkreader.org/

I think it does a great job at converting sites to dark mode, so much so that I
decided to take a little inspiration from it.

What I do is go open the site I'm working on and activeate the plugin. With my
site in a 'dark mode' from the plugin, I'll open my inspector and color drop a
color I like to get a hex code.  Firefox has a built-in color dropper, not sure
if Chrome does these days.

For these cases where you need a custom color, you can add that custom color to
the Tailwind configs

```js
module.exports = {
  theme: {
    colors: {
        wicked-dark-blue: "#00001e"
<!-- #000038 -->
    },
  },
}
```
// Create your own at: https://javisperez.github.io/tailwindcolorshades

(May have to restart everything to get it working)
(Think you need to include colors before the theme and add your colors in the
right spot.  Creating things from stock, I already had a `theme` block that I
had to add my color stuff in.  Here is what the whole config looks like now

`config/tailwind.config.js`
```js
const defaultTheme = require('tailwindcss/defaultTheme')
const colors = require('tailwindcss/colors')
// do i need colors if i don't override any?

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
      colors: {
        "wicked-dark-blue": "#000038",
        "landing-page-dark": "#19212c",
      },
    },
  },
  plugins: [
    require('@tailwindcss/forms'),
    require('@tailwindcss/aspect-ratio'),
    require('@tailwindcss/typography'),
  ],
}
```

(Maybe, looks like that actually broke a lot of shit)

You should have the new color available for use.  Remember the format for
background colors still applies: `bg-COLORNAME` so in this case it would be
`bg-wicked-dark-blue`

The same thing would go for text.  Would could update the accent color in the
hero with `text-wicked-dark-blue`

```
<span class="text-purple-600 dark:text-wicked-dark-blue">Dark</span>
```

Once you have an idea of the colors you'd like to use for your dark mode
variant, the easiest way is to add the dark variants into `@apply` and make a
class.  That's a good option once you know what you want but until then, you'll
probably spend a lot of time flipping back and forth between light and dark mode
filling in all the spots you missed.  That's normal and part of the process that
keeps it looking nicer than something that's just like a dark background with
white text.

Rails , Hotwire and Tailwind really makes me feel like I have superpowers these
days. I think this is a great feature and there are a lot of benefits to dark
mode like battery life, eye strain and badassness (debatable).  I hope this
helps and I hope this work for you.
