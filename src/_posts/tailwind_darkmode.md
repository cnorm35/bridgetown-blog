---
layout: post
title:  Adding darkmode to tailwind rails app
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

Add a hero section to your index page to get some tailwind styles going

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









