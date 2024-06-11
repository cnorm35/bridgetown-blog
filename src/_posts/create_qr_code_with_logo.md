---
layout: post
title:  Create a QR Code with a Logo
date:   2024-06-09 16:54:46 -0500
category: ruby
excerpt: "Creating a QR code with a logo is a great way to brand your QR codes and make them stand out.  This post will go over how to create a QR code with a logo using Ruby on Rails."
author: cody
---

talk about how to generate a regular one first?

use the montibus labs png

rails new qr_code_generator --css=tailwind 


be rails g scaffold QrCode url:string

root to qr_codes#index


Add Image processing

Just uncommented it in the gemfile and ran bundle install

bin/rails active_storage:install
bin/rails db:migrate

# qr_code.rb
has_one_attached :combined_image
has_one_attached :logo

update params

      params.require(:qr_code).permit(:url, :logo)


add to qr_form
  <div class="my-5">
    <%= form.label :logo %>
    <%= form.file_field :logo, class: "" %>
  </div>


display logo on show page

    <%= image_tag qr_code.logo %>


add pre-processed variants

  has_one_attached :logo do |attachable|
    attachable.variant :thumb, resize_to_limit: [100, 100]
  end

  (not sure if I need this since I'm only using the thumb in the view)

  (maybe make a qr_code (image maybe?) or something attachment to hold the QR image before the
  composite?)


# and in the view

    <%= image_tag qr_code.logo.variant(:thumb) %>



add rqr code gem gem "rqrcode", "~> 2.0"


After Commit hook and on logo or URL update?

QR codes are a great way to share links or infomation easily with your users,
unless you're a resturant, in which case you should really consider a paper menu
please.

Being able to generate a custom QR code overlayed with a logo, while still being
scannable is pretty easy with Rails.

This post will walk through setting up a simple standalone QR with logo app.

We'll be using mini magic and rqr code to create a png qr code, then create a
composite image with the logo.

