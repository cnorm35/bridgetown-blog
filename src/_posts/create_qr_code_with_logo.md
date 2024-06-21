---
layout: post
title:  Create a QR Code with a Logo
date:   2024-06-09 16:54:46 -0500
category: ruby
excerpt: "Creating a QR code with a logo is a great way to brand your QR codes and make them stand out.  This post will go over how to create a QR code with a logo using Ruby on Rails."
author: cody
---
QR codes are a great way to share links or information easily with your users,
unless you're a restaurant, in which case you should really consider a paper menu
please.

In all seriousness, it's a great way to easily let people scan and access
specific information.

To say QR codes are a dime a dozen would be an understatement.  Creating a
composite QR code with a logo is a great way to help your code stand out and add
some additional branding and feel to your app.

Being able to generate a custom QR code overlaid with a logo, while still being
scannable is pretty easy with Rails.

This post will walk through setting up a simple standalone QR with logo app.

We'll be using MiniMagick and RQRCode to create a PNG of the qr code, then create a
composite image with the logo.


### Getting started

The app is going to have a single model `QRCode`.

It will have a string field for the URL and 3 attached active storage images.

3 might be a bit overkill but this will store the logo uploaded by a user, the
QR code without the logo and finally the combined QR code image with the logo.

Create a new Rails app with Tailwind by running the following command in your
terminal.

```bash
rails new qr_code_generator --css=tailwind
```

We'll keep things simple by scaffolding the `QrCode` resources

```
bin/rails generate scaffold QrCode url:string
```

Since this is a simple app, we'll set the root route to the `QrCode` index page.

```ruby
  # config/routes.rb
  root "qr_codes#index"
```

#### Active Storage and Image Processing

With a simple model for the QR code, the next step is to add the image
processing gem and install active storage for the images that will be attached.

If you look in the `Gemfile`, you should already see the `image_processing` gem
in your `Gemfile`

```ruby
# Use Active Storage variants [https://guides.rubyonrails.org/active_storage_overview.html#transforming-images]
# gem "image_processing", "~> 1.2"
```

You can uncomment the gem and run `bundle install`

With the image processing gem installed and ready to go, the next step is to
install Active Storage and run the corresponding migrations.

```
bin/rails active_storage:install
bin/rails db:migrate
```

With Active Storage installed, we can add our attachments we'll be using.

`app/models/qr_code.rb`
```ruby
class QrCode < ApplicationRecord
  has_one_attached :logo
  has_one_attached :original_image
  has_one_attached :combined_image
end
```

We'll be generating the `combined_imgae` and the `original_image` based on
inputs from the user, including a `logo` file.


Updating the form and adding the file attributes to the params in the controller
will allow us to start storing the uploaded logo and URL attached to the QR
code.

Add the file input for the logo to the form.

I'm also limiting the file type to PNG for simplicity.  Or QR codes are going to be PNGs, so it makes sense to limit the logo to the same format. Accepting PNGs with transparent backgrounds also makes it easier to overlay the logo on the QR code after some processing.

This is by no means a foolproof way to ensure the file is a PNG, but it's a good start. If you find yourself needing more, you can look into validating the file type in the model before saving.

`app/views/qr_codes/_form.html.erb`

```ruby
  <div class="my-5">
    <%= form.label :logo %>
    <%= form.file_field :logo, accept: "image/png" %>
  </div>
```

Add `logo` to the strong params in the controller.

`app/controllers/qr_codes_controller.rb`
```ruby
    def qr_code_params
      params.require(:qr_code).permit(:url, :logo)
    end
```

Adding the logo to the show page will give us an easy way to reference the image
we uploaded.

`app/views/qr_codes/_qr_code.html.erb`

```ruby
<div id="<%= dom_id qr_code %>">
  <p class="my-5">
    <strong class="block font-medium mb-1">Url:</strong>
    <%= qr_code.url %>
    <%= image_tag qr_code.combined_image %>
  </p>
</div>
```

If you haven't started the server yet, now would be a good time to start everything
up and make sure it's working.

If things are good to go, you should land directly on the `QrCode` index page.

Clicking the New QR Code button, filling out the URL, and uploading a logo should create a new QR code record with the URL and logo attached.

### QR codes.

We'll be using the [rqrcode](https://github.com/whomwah/rqrcode){:target="_blank"} gem to generate QR codes as a PNG and saving as an
Active Storage attachment.

Add it to the Gemfile with::

```ruby
gem "rqrcode", "~> 2.0"
```
and install the gem with `bundle install`

To keep a lot of the image processing logic out of the controller, we'll create a method on the model to generate the QR code and save it as an attachment.

`app/models/qr_code.rb`

```ruby
  def generate_qr_code!
    qr = RQRCode::QRCode.new(url)
    original_qr = qr.as_png(size: 400,
      bit_depth: 1,
      border_modules: 4,
      color_mode: ChunkyPNG::COLOR_GRAYSCALE,
      color: "black",
      file: nil,
      fill: "white")
    original_image.attach(io: StringIO.new(original_qr.to_s),
                          filename: "original_qr.png",
                          content_type: "image/png")
    save!
  end
```

This method will generate a QR code based on the URL and save it as an attachment.

Another advantage of having the QR code generation on the model is that we can
test and diagnose issues in our Rails console without needing to go through the
controller.

Running `qr_code.generate_qr_code!` in the console (or anywhere) should generate a QR code
from the URL and save it as an attachment.

Calling this method after we've successfully saved the `QrCode` record will create the QR code and save it as an attachment.

`app/controllers/qr_codes_controller.rb`

```ruby
  def create
    @qr_code = QrCode.new(qr_code_params)

    respond_to do |format|
      if @qr_code.save
        @qr_code.generate_qr_code!
        format.html { redirect_to qr_code_url(@qr_code), notice: "Qr code was successfully created." }
        format.json { render :show, status: :created, location: @qr_code }
      else
        format.html { render :new, status: :unprocessable_entity }
        format.json { render json: @qr_code.errors, status: :unprocessable_entity }
      end
    end
  end

```

So we can make sure the image was created an attached, we can display it on the
details page.

`<%= image_tag qr_code.original_image %>`

`app/views/qr_codes/_qr_code.html.erb`

```ruby
<div id="<%= dom_id qr_code %>">
  <p class="my-5">
    <strong class="block font-medium mb-1">Url:</strong>
    <%= qr_code.url %>
    <%= image_tag qr_code.logo %>
    <%= image_tag qr_code.original_image %>
  </p>
</div>
```

If everything is working as expected, you should see the QR code image on the show page for the QR code record.
To check to make sure everything is working as expected, you can call
`generate_qr_code!` on an existing record in the Rails console or create a new record through the form.
(Screenshot of the QR code on the show page - Use real URLs)

### Going Further

Adding a bit of flair to things, we're ready to start combining the uploaded
logo, this will be the `combined_image` attachment.

At this point, there are 2 attached image to the QR code model, `original_image`
and `logo`.

We're now going to use [MiniMagick](https://github.com/minimagick/minimagick){:target="_blank"} to do a bit of formatting and create a
composite image.

The combined logo will be using the same approach as the original QR code,
meaning this code will be added to a method in the `QrCode` model.

To start, we create a `MiniMagick::Image` for the `logo` and `original_image` qr code.

Before attempting to combine the images, we'll need to do some processing on the
logo image.  This will include resizing the image and adding a white background to make the logo stand out from the QR code.

Since we're going to be doing some image processing, we'll need to require the `mini_magick` gem in the `QrCode` model.

First, we create a `MiniMagick::Image` object for the logo image.

```ruby
logo_image = MiniMagick::Image.open(logo)
```
Then we do the same for the QR code image.

```ruby
qr_code_image = MiniMagick::Image.open(original_image)
```

Next, we'll resize the logo image and add a white background to help the logo stand out.

```ruby
updated_logo_image = ImageProcessing::MiniMagick.source(logo_image)
  .resize_and_pad(100, 100, background: "white").call
```

After creating the updated logo image, we can create a composite image with the QR code and logo.

```ruby
composite_qr_code = qr_code_image.composite(updated_logo_image) do |c|
  c.compose "Over"
  c.gravity "center"
  c.colorspace "sRGB"
end
```

The `composite` method is used to overlay the updated logo image on top of the QR code image.  The `compose` option is set to "Over" to overlay the logo on top of the QR code.  The `gravity` option is set to "center" to center the logo on the QR code.  The `colorspace` option is set to "sRGB" to ensure the colors are displayed correctly.

Finally, we'll attach the composite image to the `combined_image` attachment.

```ruby
qr_code.combined_image.attach(filename: "combined_qr_code.png",
                io: File.open(composite_qr_code),
                content_type: "image/png")

```

Then create the composite image and stores it as an Active Storage attachment.

Here's what the final method looks like in the `QrCode` model.

```ruby
  def generate_combined_image!
    logo_image = MiniMagick::Image.open(logo)
    qr_code_image = MiniMagick::Image.open(original_image)

    updated_logo_image = ImageProcessing::MiniMagick.source(logo_image).resize_and_pad(100, 100, background: "white").call

    composite_qr_code = qr_code_image.composite(updated_logo_image) do |c|
      c.compose "Over"
      c.gravity "center"
      c.colorspace "sRGB"
    end

    combined_image.attach(filename: "combined_qr_code.png",
                          io: File.open(composite_qr_code.path),
                          content_type: "image/png")
    save!
  end

```

If you'd like to break this down into smaller steps, you can call each method individually in the Rails console.  Here's an example of how you could call each method individually in the Rails console to create the composite image and save it as an attachment.

``` ruby
# in the Rails console
require "mini_magick"
qr_code = QrCode.last
logo_image = MiniMagick::Image.open(qr_code.logo)
qr_code_image = MiniMagick::Image.open(qr_code.original_image)
updated_logo_image = ImageProcessing::MiniMagick.source(logo_image)
  .resize_and_pad(100, 100, background: "white").call
composite_qr_code = qr_code_image.composite(updated_logo_image) do |c|
  c.compose "Over"
  c.gravity "center"
  c.colorspace "sRGB"
end
qr_code.combined_image.attach(filename: "combined_qr_code.png",
                        io: File.open(composite_qr_code.path),
                        content_type: "image/png")
qr_code.save!
```


In the same way we called `generate_qr_code!` after saving the record, we can call `generate_combined_image!` to create the composite image with the logo from the Rails console on an existing record that has a logo and QR code attached.

The last step is to call the `generate_combined_image!` method after the QR code has been generated and saved.


`app/controllers/qr_codes_controller.rb`

```ruby
  def create
    @qr_code = QrCode.new(qr_code_params)

    respond_to do |format|
      if @qr_code.save
        @qr_code.generate_qr_code!
        @qr_code.generate_combined_image!
        format.html { redirect_to qr_code_url(@qr_code), notice: "Qr code was successfully created." }
        format.json { render :show, status: :created, location: @qr_code }
      else
        format.html { render :new, status: :unprocessable_entity }
        format.json { render json: @qr_code.errors, status: :unprocessable_entity }
      end
    end
  end
```

We'll also include this in the update action so that the combined image is updated whenever the QR code is updated.

```ruby
  def update
    respond_to do |format|
      if @qr_code.update(qr_code_params)
        @qr_code.generate_qr_code!
        @qr_code.generate_combined_image!
        format.html { redirect_to qr_code_url(@qr_code), notice: "Qr code was successfully updated." }
        format.json { render :show, status: :ok, location: @qr_code }
      else
        format.html { render :edit, status: :unprocessable_entity }
        format.json { render json: @qr_code.errors, status: :unprocessable_entity }
      end
    end
  end
```

Display the combined image on the show page for the QR code record to see the final result.

`<%= image_tag qr_code.combined_image %>`

`app/views/qr_codes/_qr_code.html.erb`

```ruby
<div id="<%= dom_id qr_code %>">
  <p class="my-5">
    <strong class="block font-medium mb-1">Url:</strong>
    <%= qr_code.url %>
    <%= image_tag qr_code.logo %>
    <%= image_tag qr_code.original_image %>
    <%= image_tag qr_code.combined_image %>
  </p>
</div>
```


If all goes as planned, you should be re-directed to the show page for the newly
created QR Code record and see if all the logos are correct.

<div class="my-5">
<img
    alt="Combined QR Code with Logo"
    class="position-relative mx-auto rounded w-100 shadow-lg"
    src="/images/CombinedQRCodeExample.png"
    style="z-index: 10"
/>
</div>


And with that, you've created a simple tool too add some style and flair to your
QR codes.

This is a pretty simple example and there are a lot of ways to improve and expand on this feature.  You could add more options for the QR code generation, add more image processing options, or even add a background color to the QR code for some additional flair.

You can view the full code for this example on
[GitHub](https://github.com/cnorm35/qr_code_generator_example){:target="_blank"}.
