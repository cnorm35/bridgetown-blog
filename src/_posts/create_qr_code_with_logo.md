---
layout: post
title:  Create a QR Code with a Logo
date:   2024-06-09 16:54:46 -0500
category: ruby
excerpt: "Creating a QR code with a logo is a great way to brand your QR codes and make them stand out.  This post will go over how to create a QR code with a logo using Ruby on Rails."
author: cody
---

<!-- use the montibus labs png -->

<!-- After Commit hook and on logo or URL update? -->

QR codes are a great way to share links or infomation easily with your users,
unless you're a resturant, in which case you should really consider a paper menu
please.

In all seriousness, it's a great way to easly let people scan and access
specific information.

To say QR codes are a dime a dozen would be an understatement.  Creating a
composite QR code with a logo is a great way to help your code stand out and add
some additional branding and feel to your app.

Being able to generate a custom QR code overlayed with a logo, while still being
scannable is pretty easy with Rails.

This post will walk through setting up a simple standalone QR with logo app.

We'll be using mini magic and rqr code to create a png qr code, then create a
composite image with the logo.


### Getting started

Create a new Rails app with Tailwind (optional).


```ruby
rails new qr_code_generator --css=tailwind
```

cd into the directory for the app and commit your changes.

Our QR logo app is going to have a single model `QRCode`

It will have a string field for the URL and 3 attached active storage images.

3 might be a bit overkill but this will store the logo updloaded by a user, the
QR code without the logo and finally the combined QR code image with the logo.

We'll keep things simple by scaffolding the `QRCode` model


```
bin/rails generate scaffold QrCode url:string
```

One last thing to update before moving on is to set our root route to the QRCode
index

```ruby
  # config/routes.rb
  root "qr_codes#index"
```


With a simple model for the QR code, the next step is to add the image
processing gem and install active storage for the images that will be attched.

If you look in your `Gemfile`, you should already see the `image_processing` gem
commented out. You can uncomment the gem and run `bundle install`

With the image processing gem installed and ready to go, the next step is to
install ActiveStorage and run the cooresponding migrations.

```
bin/rails active_storage:install
bin/rails db:migrate
```

With Active Storage installed, we can add our attachments we'll be using.

```ruby
# qr_code.rb
class QrCode < ApplicationRecord

  has_one_attached :combined_image
  has_one_attached :logo
  has_one_attached :original_image
  ...
end
```


We'll be generating the `combined_imgae` and the `original_image` based on
inputs from the user, including a `logo` file.


Updating the form and adding the file attributes to the params in the controller
will allow us to start storing the uploaded logo and and URL attached to the QR
code.

Add the file input for the logo to the form

`app/views/qr_codes/_form.html.erb`

```ruby
  <div class="my-5">
    <%= form.label :logo %>
    <%= form.file_field :logo, class: "" %>
  </div>
```

Whitelist the logo in the controller

`app/controllers/qr_codes_controller.rb`
```ruby
    def qr_code_params
      params.require(:qr_code).permit(:url, :logo)
    end
```


Adding the logo to the show page will give us an easy way to reference the image
we uploaded.

`app/views/qr_codes/_qr_code.html.erb`
```
<div id="<%= dom_id qr_code %>">
  <p class="my-5">
    <strong class="block font-medium mb-1">Url:</strong>
    <%= qr_code.url %>
    <%= image_tag qr_code.combined_image %>
  </p>
</div>
```


If you haven't started the server, now would be a good time to start everything
up and make sure it's working.

If things are good to go, you should land directly on the QrCodes index page.
Clicking the New QR Code button, filling out the URL, (validations?) and
uploading a logo.

### QR codes.

We'll be using the rqrcode gem to generate QR codes as a PNG and saving as an
ActiveStorage attchment.

https://github.com/whomwah/rqrcode

Add the gem to the gemfile

```ruby
gem "rqrcode", "~> 2.0"
```

and install the gem with `bundle install`

```ruby
code to create and save a QR code

put in a method on the model?

    qrcode = RQRCode::QRCode.new(url)
    original_qr = qrcode.as_png(size: 400,
      bit_depth: 1,
      border_modules: 4,
      color_mode: ChunkyPNG::COLOR_GRAYSCALE,
      color: "black",
      file: nil,
      fill: "white")

original_image.attach(
  filename: "qr_code",
  io: StringIO.new(original_qr.to_s),
  content_type: "image/png"
)
original_image.save
    logo_image = MiniMagick::Image.read(logo.download)
    qr_code_image = MiniMagick::Image.read(original_image.download)

    # add white background to png
    updated_logo_image = ImageProcessing::MiniMagick.source(logo_image).resize_and_pad!(100, 100, background: "white")

        composite_qr_code = ImageProcessing::Vips.source(qr_code_image).composite(updated_logo_image, offset: [150,150]).call
    combined_image.attach(filename: "combined_qr_code.jpg",
                          io: File.open(composite_qr_code),
                          content_type: "image/jpg")


```


### Going Further

Adding a bit of flair to things, we're ready to start combining the uploaded
logo, this will be the `combined_image` attachment.


I'm not claiming this is the easiest or the most performant way but the best way
to get a good answer on the internet is to publicaly proclaim a wrong one.

This started out as a fun feature I was exploring and wanted to get a proof of
concept working.  I didn't end up using the feature, but thought it was a really
cool feature.

At this point, there are 2 attached image to the QR code model, `original_image`
and `logo`

We're now going to use MiniMagick to do a bit of formatting and create a
composite image

To start, we create a MiniMagick::Image for the `logo` and `original_image` qr
code.

The first step is to resize the logo image and add a white background to help
our logo stand out. 


```ruby
    logo_image = MiniMagick::Image.read(logo.download)
    qr_code_image = MiniMagick::Image.read(original_image.download)

    # add white background to png
    updated_logo_image = ImageProcessing::MiniMagick.source(logo_image).resize_and_pad!(100, 100, background: "white")
```

Then create the composite image and store as an Active Storage attachment.

```ruby
    composite_qr_code = ImageProcessing::Vips.source(qr_code_image).composite(updated_logo_image, offset: [150,150]).call
    combined_image.attach(filename: "combined_qr_code.jpg",
                          io: File.open(composite_qr_code),
                          content_type: "image/jpg")
```

If all goes as planned, you should be re-directed to the show page for the newly
created QR Code record and see if all the logos are correct

(screenshot and example QR)


And with that, you've created a simple tool too add some style and flair to your
QR codes.


