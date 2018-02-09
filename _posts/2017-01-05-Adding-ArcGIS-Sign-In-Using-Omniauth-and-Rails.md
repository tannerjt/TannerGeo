---
layout: post
title:  "Adding ArcGIS Sign In Using Omniauth and Rails"
author: Joshua Tanner
description: Sign into your Ruby on Rails app with an ArcGIS login.
image: assets/images/door.jpg
---

![test image]({{ site.url | absolute_path}}/assets/images/door.jpg)

We're all familiar with being able to sign into other applications with our Facebook
or Google account, but what about an ArcGIS Online account? This is possible because
ArcGIS Online uses OAuth 2 as it's authentication framework.  There is limited information on
how this process works, so this post is meant to shed some light into the process.  

For this tutorial, we will create a dummy application that will allow users to sign in using their existing
ArcGIS Online credentials.  Once signed in, the user will be able to post a message. Pretty simple!  

The process assumes you have a working knowledge of Ruby on Rails and a basic understanding of OAuth and authentication
workflows in Rails.


## Step 1. Application Setup

We must first create our rails application using the command line rails generator:

```
rails new MySecretApp
cd MySecretApp
```

## Step 2. Generate Models

Our basic application only needs two data models; `User` and  `Message`.  Let's
create the `message` model:

```
rails generate scaffold message body:text
rils db:migrate
```  

We used `generate scaffold` because we also want the controller and corresponding views
to be created automatically.

Our message currently only has one field (body) of type text.  This will be the body of the
message that an authenticated user will will write.  

Next, we need to create our user model.  To do that, we will use a gem called Devise, which will
also handle our authentication workflows.  Place the following gem in your Gemfile:

```
gem 'devise', '~> 4.2'
```

and run `bundle install`

Run the Devise generator:

```
rails genererate devise:install
```

We need to setup the default URL for our Devise mailer in our development environment:

```
# /config/environments/development.rb
config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
```

Lastly, we need to generate our `user` model using a Devise generator.  This will create
all of the necessary fields for handling authentication.

```
rails generate devise user
```

To persist our changes to the database, run the database migration

```
rails db:migrate
```

## Step 3. Configure Associations

For active record to know about the association between our user and their corresponding messages,
we need to add the association to our models:

A message belongs_to a user:

```
# /app/models/message.rb
class Message < ApplicationRecord
  belongs_to :user
end
```

A user has many message.  If a user is deleted, all dependent messages are also removed:

```
# app/models/user.rb
class User < ApplicationRecord
  has_many :messages, dependent: :destroy
  ...
end
```

To associate a message with a particular user, we need to add the user as a foreign key
to the message model:

```
rails generate migration add_user_to_message user:references
rails db:migrate
```

## Step 4. Wiring Everything Together

So far, we've added our message scaffold, created an authenticated user model, and hooked up the
associations for our active directory to work as intended.  This was all done with the help
of Rails generators, and the Devise authentication gem.  Now, we need to tie all these parts
together for a working application.

When a user visits the site, the home page should show a list of posted messages, which
requires us to set the root route:

```
# config/routes.rb
...
root 'messages#index'
```

Next, we need to modify our app to accomodate the security measures created by adding our Devise user
model.  First, let's restrict the ability to create, update, or delete messages unless a user is
signed in:

```
# /app/controllers/messages_controller.rb
before_action :authenticate_user!, except: [:index, :show]
```

This limits any unauthenticated user to only access the index and show actions of the messages
controller.  Next, we need to modify some of the actions to associate the messages with the current
user.  Modify the actions in the messages controller:

```
# /app/controllers/messages_controller.rb
def new
  @message = current_user.messages.build
end
def create
  @message = current_user.messages.build(message_params)
end
```

Lastly, we need to add some navigation links to our main application template for signing in and
signing out `app/layouts/application.html.erb`:

```
<body>
  <% if user_signed_in? %>
    <%= link_to "Sign Out", destroy_user_session_path, method: :delete %>
  <% else %>
    <%= link_to "Sign In", new_user_session_path %>
    <%= link_to "Sign Up", new_user_registration_path %>
  <% end %>
  <%= yield %>
</body>
```

Now we have a working application that has built in authentication.  Give it a try by running
`rails s` and testing your app.

## Step 5. Sign in With ArcGIS

When we're signing in now, were doing so by creating a new account.  What we want to do, is avoid this by
allowing users to sign in with their existing ArcGIS Online accounts.  Devise supports this by adding
OmniAuth.  The first step is to add the ArcGIS OmniAuth and support for OAuth2 gem to our Gemfile:

```
gem 'omniauth-oauth2', '~> 1.3.1'
gem 'omniauth-arcgis', '~> 0.1.1'
```

Install gems:

```
bundle install
```

We need to update the user model to accomodate the ArcGIS Provider and user id:

```
rails generate migration add_omniauth_to_users provider:string uid:string
rails db:migrate
```

In order to enable OAuth with ArcGIS Online, you need to register the application
on [http://developers.arcgis.com](http://developers.arcgis.com).  After you have registered
the application, add your development URL (http://localhost:3000) to the Redirect URI form
under the authentication tab.  Notice the Client and Client Secret listed on the page?  We will
use these to setup our OmniAuth.

While you could hard code the Cleint ID an Secret into your initializer, I like to use
[Figaro](https://github.com/laserlemon/figaro) to manage the setting of my environment variables.  

```
gem 'figaro', '~> 1.1', '>= 1.1.1'
```

```
bundle install
bundle exec figaro install
```

This adds a new file `config/application.yml` and adds it to our `.gitignore` file.  Add your Client ID
and Secret here:

```
arcgis_client_id: 'xxxxx'
arcgis_client_secret: 'xxxxx'
```

This will allow us to access ENV['arcgis_client_id'] from anywhere in our application.

Next, declare the ArcGIS provider in `config/initializers/devise.rb`

```
config.omniauth :arcgis, ENV['arcgis_client_id'], ENV['arcgis_client_secret']
```

Make our user model omniauthable:

```
# /app/models/user.rb
devise :omniauthable, :omniauth_providers => [:arcgis]
```

Setup our omniauth callback URL in our routes `config/routes.rb`

```
devise_for :users, :controllers => { :omniauth_callbacks => "omniauth_callbacks" }
```

Add the file for our OmniAUth callbacks that inherits from Devise `app/controllers/omniauth_callbacks_controller`

```
class OmniauthCallbacksController < Devise::OmniauthCallbacksController
  def arcgis
    @user = User.from_omniauth(request.env["omniauth.auth"])

    # sign_in_and_redirect @user
    if @user.persisted?
      sign_in_and_redirect @user, :event => :authentication
      set_flash_message(:notice, :success, :kind => "arcgis") if is_navigational_format?
    else
      session["devise.arcgis_data"] = request.env["omniauth.auth"]
      redirect_to new_user_registration_url
    end
  end

  def failure
    redirect_to root_path
  end
end
```

Lastly, we need to add the above `from_omniauth` method to our user model

```
def self.from_omniauth(auth)
  where(provider: auth.provider, uid: auth.uid).first_or_create do |user|
    user.email = auth.info.email
    user.password = Devise.friendly_token[0,20]
  end
end
```

And that's it! This is a very simple applcation, but it shows how we can use the OAuth2 capabilities of ArcGIS Online to sign into our app.  
