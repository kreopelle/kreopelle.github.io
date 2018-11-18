---
layout: post
title:      "React/Redux Final Project"
date:       2018-11-16 01:59:54 -0500
permalink:  react_redux_final_project
---


This is my attempt to piece together advice across the Internet* to write a tutorial for using ActiveStorage with React. Follow these steps to upload any file, be it mp3, jpeg or pdf, from a form on a React/Redux application to Active Storage on a Rails API. 

This tutorial uses a local storage system and does not use direct uploads. I hope to write addendums to this post to accommodate those processes as I understand them better.

There are tons of awesome tutorials that go in depth on setting up a React App with a Rails API on the backend. I got started using the guides by [Full Stack React](https://www.fullstackreact.com/articles/how-to-get-create-react-app-to-work-with-your-rails-api/) and [Nick Hartunian](https://medium.com/@nick.hartunian/rails-5-api-create-react-app-full-stack-heaven-2b2160b5ce6b). 

After starting your basic React App with a Rails API, we’ll:

- Install ActiveStorage
- Set up a model, controller, and serializer to handle file attachments
- Create stateful React components connected to the Redux store to upload and display your content
- Generate reducers and actions to make asynchronous requests to your Rails API


Here’s a guide to fast-forward through the setup:

```
$ rails new app_name --api  
$ cd app_name 
$ create-react-app client 
```

To create a rake task that will start both servers at once:  

* Add Foreman to your Gemfile and run bundle install 

```
# Gemfile 
gem ‘foreman’
```
```
$ bundle install 
```

* Create a Procfile 

```
$ touch Procfile
``` 

* Add to the file:  

```
# Procfile

web: sh -c ‘cd client && npm start’
api: bundle exec rails s -p 3001
```

* Create a new rake task to run that command: 

```
$ touch lib/tasks/start.rake 
``` 

* And in that file, paste: 

```
# lib/tasks/start.rake

task :start do
  exec 'foreman start -p 3000'
end 
``` 

Now you have the basic skeleton for your app as well as a command to start both your Rails API (located at `localhost:3001`) and your React app (located at `localhost:3000`) simultaneously. Just type: 

```
$ rake start
```

Beautiful! You should see the spinning React logo open in a browser window. If you navigate to localhost:3001, you should be greeted by our Rails cartoon friends.

Now for the fun stuff!

### Install active_model_serializers gem

This gem prepares Model attributes to be rendered into JSON. Down the line, we’ll use it to include the url for our attached file in the JSON representation of our model. Add it to your Gemfile and run bundle install.

```
# Gemfile

gem ‘active_model_serializers’
```
```
$ bundle install 
```

**NOTE: ** Active Model Serializers is, at the time of writing, undergoing renovations. Rails may have other approved methods/processes in the future.   

### Create the model you’d like to attach a file to

For the sake of this tutorial, we’ll run a scaffold generator for our model. This will create a controller with actions ready to render JSON (thank you API mode!), a model, a serializer with attributes pre-filled, and a migration ready to run for our DB. 

``` $rails g scaffold posts title:string description:string
```

After the generator is finished, check out your files to make sure they’re what you hope they’d be. If all’s well, migrate the database. 

```
$ rails db:migrate
```  
### Install Active Storage 

If you’re new to Active Storage, a tool that facilitates attaching files to Active Record models, I highly recommend you check out the [Active Storage Overview on Rails Guides](https://edgeguides.rubyonrails.org/active_storage_overview.html). Previously gems like Paperclip facilitated attached files, but as of Rails 5.2, Active Storage comes ready to install with any Rails app.

To install, run: 

```
$ rails active_storage:install 
```
This will generate two tables in your application’s database, `active_storage_blobs` and `active_storage_attachments`. Previous solutions required columns to be added to existing models to accommodate attachments. 

Instead, Attachment is a join model that connects Blobs (which stands for Binary Large OBject) to your models. 

According to [Evil Martians](https://evilmartians.com/chronicles/rails-5-2-active-storage-and-beyond), `active_storage_blobs` don’t put the binary into your database, but tracks the location of the binary file and its associated metadata. 

### Associate Model, Controller, and Serializer with File

* Model: 

To associate a file with your model, you just need to add `has_one_attached` and then the attribute name for that file to your model. The attribute name can be anything you’d like.

```
# app/models/post.rb

class Post < ApplicationRecord

has_one_attached :file

end
```
If you’d like to associate multiple files with an Active Record Model, you can use `has_many_attached` instead. I haven’t tested the rest this tutorial using the `has_many_attached` association.

* Controller: 
Add the attribute assigned to has_one_attached from your model to the private params method at the bottom of your controller. 

```
#app/controllers/posts_controller.rb 

… 

private 

def post_params
  params.require(:post).permit(:title, :body, :file)
end 

… 
``` 

* Serializer: 

Right now your file will exists as a blob, but to use it in your React app, we need to serialize the url that points to where this blob lives in your database (remember, to your program it is just a large binary object). To make this happen we need to include Rails’ url_helpers and write a method that will return the associated blob url. 

According to the [Rails API](https://api.rubyonrails.org/v5.1/classes/ActionDispatch/Routing/UrlFor.html), url_helpers enable, among other things, access to those handy prefix methods like `posts_path`. In our case, we’d like to get access to the blob url associated with our file. 

Those route methods are automatically included in controllers, views, and mailers. To access them in other directories, they’ll need to be explicitly included. Just below the class definition for your serializer, write:

``` 
# app/serializers/post_serializer.rb 

class PostSerializer < ActiveModel::Serializer

# enable access to the url helpers in the serializer 
  include Rails.application.routes.url_helpers 

  attributes :id, :title, :body

end

``` 
Next, write a method that creates an attribute pointing to the url related to your blob file. Use the rails_blob_url helper method to generate a permanent link to the resource, and add the method’s name to the list of attributes:

``` 
#app/serializers/post_serializer.rb 

class PostSerializer < ActiveModel::Serializer 
   include Rails.application.routes.url_helpers

  attributes :id, :title, :description, :file_url

  def file_url
    return rails_blob_url(object.file)
  end 

end 
``` 
This won’t work out of the box, as you need to provide a `default_url` option to tell your Rails app what the prefix for the `blob_url` should be. 

### Configure your application

Navigate to config/environments/development.rb. This file holds configuration for your application in development mode. When you transfer the application to production mode, you’ll need to repeat a similar process in the config/environments/production.rb file. 

After the closing `end` statement for `Rails.application.configure`, add the following: 

``` 
# config/environments/development.rb 

Rails.application.routes.default_url_options[:host] = “localhost:3001” 

``` 

This line sets the default host for your `url_helpers`. The `url_helpers` generate the end of the path for your application, not the hosting information. By default, Rails sets the host as `localhost:3000`, but that won’t work because we’re running our React app on that port. Instead, we need to explicitly set this to `localhost:3001` to generate the correct host information in the url for our attached file. 

While we’re configuring things, let’s enable rack-cors. This gem allows our Rails app to accept Cross-Origin-Resource-Sharing requests (cors) from our React app, so we can make asynchronous javascript requests (also known as AJAX) to our Rails API. 

Hop over to your Gemfile, uncomment and install the rack-cors gem. 

```
# Gemfile 

gem ‘rack-cors’

# and in your terminal, run 

$ bundle install

```

Then head to `config/application.rb`. We’ll need to configure Rack::Cors to accept requests from the React app’s origin. Within the class definition for the Rails application, add: 

```
# config/application.rb 

… 

module YourApp
  Class Application < Rails::Application 
    …
    
    config.api_only = true 

    #=> Middleware to enable cross-origin requests 
    config.middleware.insert_before 0, Rack:Cors do
      allow do
        origins ‘http://localhost:3000' #=> or whatever host your React app points to
        resource ‘*’, :headers => :any, :methods, => [:get, :post, :options]
      end 
    end 

  end 
end 

``` 

This middleware explicitly allows any requests from `localhost:3000` to be accepted by our Rails API. 

### YOUR RAILS API IS OFFICIALLY READY FOR LIFTOFF 

Take a brief intermission before we dive into the React portion. Perhaps, by watching this lovely video: 




<iframe src="https://player.vimeo.com/video/27315673" width="640" height="360" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
<p><a href="https://vimeo.com/27315673">Trim</a> from <a href="https://vimeo.com/peteyboy">Peter Simon (Petey Boy)</a> on <a href="https://vimeo.com">Vimeo</a>.</p>




Okay, now we’re back. It’s React time.

For brevity’s sake, we’re going to just use the pre-made App component for our own devices. To truly follow React’s presentational/container pattern and take advantage of the beauty of components for a single purpose, I would recommend creating separate components for: 
the form that holds the upload field
the container that displays the content from the api
the individual records retrieved from the api

If you haven’t already, check out [Thinking in React](https://reactjs.org/docs/thinking-in-react.html) to get up to speed on the process. Long story short (but hopefully not made longer by this intermission), we’re skipping best practices and encouraged design patterns to get to what’s necessary to make your uploads happen. 

At this point, you’ve gone through the hard and cryptic stuff. The rest is just building a React application with a Redux store that uses Thunk middleware to make POST and GET requests to your Rails API. 

