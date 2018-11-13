---
layout: post
title:      "Revamp your existing Rails app to use a JavaScript frontend "
date:       2018-11-11 13:49:13 -0500
permalink:  revamp_your_existing_rails_app_to_use_a_javascript_frontend
---


This is the first part of a tutorial that covers how to transition an existing Rails app to use a Javascript frontend without using `remote: true`. It walks through the process to load content for index and show actions in a single model with Javascript.

This first part outlines the process for setting up the requirements for your application to respond to jQuery and access your data in JSON. The second part will focus on painting the DOM using event listeners and asynchronous requests.

### Duplicate the repository that holds your Rails app. 

To duplicate the repository, I followed the method to mirror a repository as listed on [GitHub Help](https://help.github.com/articles/duplicating-a-repository/). 

First, [create a new repository on Github](https://help.github.com/articles/creating-a-new-repository/) for your Rails/JS app to live in. I named this repo after my Rails app with “-js” tacked on the end. 

Next, open your terminal, and create a “bare clone” of the repository that holds your existing Rails app using the `https://` URL. 


```$ git clone --bare https://github.com/yourusername/existing-rails-repository.git```


If you look at the new folder that’s created, it doesn’t include a lot of human-meaningful information, or resemble much of what you might expect the repository to look like at all. 

My understanding of this bare clone is a GitHub skeleton copy of the repository, just basic, under the hood stuff that makes up the structure of the repo – stuff you probably didn’t create. 

`cd` into this skeleton copy. 

```$ cd existing-rails-repository.git ```

The next step is where the magic happens: 

```$ git push --mirror https://github.com/yourusername/new-repository-js.git``` 

This command pushes a "mirror" of your existing repository to the new respository we created to house the JS-ified version of your Rails app. 

Think of a `mirror push` as copying and pasting everything from your old repo–the diretory structure, files, commits, branches, etc.–to the new repository. You may have to log in to your GitHub account via the command line to make this work.

Visit the page that houses your new repository to check it out. You should see a perfect copy of the repository that housed your Rails app! Visit the page for your Rails app’s repository, and you should see nothing changed there either!

Let’s clean up our work. Delete the bare clone copy of your old repo:

```
$ cd ..
$ rm -rf existing-rails-repository.git
``` 

Now you have a fresh copy of your Rails repo to start adding JavaScript to. Clone down a copy of this repository onto your device and let's start preparing it to handle your new JS code. 

```
$ git clone git@github.com:yourusername/new-repository-js.git
$ cd new-repository-js
```


### Add jquery-rails and active_model_serializers to your Gemfile

The following gems help to make your transition a lot easier: 

* **[jquery-rails](https://github.com/rails/jquery-rails)**: provides access to [jQuery](https://jquery.com/), a Javascript Library that simplifies DOM manipulation and AJAX requests, and the [jQuery UJS](https://robots.thoughtbot.com/a-tour-of-rails-jquery-ujs) adapter, enables JavaScript functionality for browsers that support it without negatively impacting browsers that don’t.

* **[active_model_serializers](https://github.com/rails-api/active_model_serializers/blob/v0.10.6/docs/general/getting_started.md)**: provides a Rails-y way to facilitate converting models into JSON by specifying attributes and relationships that should be present in the JSON version of your data. 

In your Gemfile, add `active_model_serializers` and `gem ‘jquery-rails’`. Run bundle install.

``` 
gem ‘jquery-rails’ 
gem ‘active_model_serializers’
``` 

### Add jquery and jquery_ujs to your Asset Pipeline

Rails optimizes the use of Javascript and CSS through the asset pipeline. According to [RailsGuides](https://guides.rubyonrails.org/asset_pipeline.html), The three main features of the pipeline are: 

1. Concatenate assets into one master .js and one master .css file to allow the browser to make one request for all JS and all CSS content (fewer requests —> faster loading)

2. Minify/compress files, moving them from a human-friendly format to the smallest possible version (smaller files —> faster loading) 

3. Allow assets to be written in higher-level languages, such as Sass for CSS and CoffeeScript for JavaScript 

`application.js` is a manifest file–a file that tells the Asset Pipeline which files it should include when it makes the minified, concatenated JS file to send to your browser. This works through “directives”, a functionality enabled by the [Sprockets gem](https://github.com/rails/sprockets), the gem that powers the asset pipeline. The syntax for a directive is: 

``` //= require your_js_file ``` 

This is where all JS requirements, including libraries (such as jQuery) and your own JavaScript files, should be listed. 

Add to `app/assets/javascripts/application.js`: 

```
//= require jquery
//= require jquery_ujs 
```

While we’re here, let’s make sure `//= require_tree` is both present and the very last line in the file. 

`require_tree` adds all JS files located within the `assets/javascripts` directory to the application.js manifest. See the [Rails Guide on the Asset Pipeline](https://guides.rubyonrails.org/asset_pipeline.html#manifest-files-and-directives) to learn about how to add files from other directories to your `application.js` manifest. 

The directives are processed from top to bottom, so make sure the external libraries your js files depend on are listed before require_tree or any other js file that may require them. 

The JavaScript manifest file is pre-loaded into your html when you create a Rails app using the `rails new app_name` command. Nested within the `<head>` tag in your `app/views/layouts/application.html.erb` file, you should see: 

```
<%= javascript_include_tag 'application', 'data-turbolinks-track': 'reload' %>
``` 

The `javascript_include_tag ‘application’` part of this line of code is telling the browser to load the `application.js` file, enabling all of the required JavaScript listed within it to be loaded as well. 

### Create your serializer(s)

Active Model Serializers has a generator to create a serialize for an existing model: 

`rails g serializer your_model`

This will create a new `serializers` directory within the `app` directory, and a new file, `your_model_serializer.rb`. The contents of the file should look something like:

``` 
class YourModelSerializer < ActiveModel::Serializer
  attributes :id
end
```

The default generated serializer includes just the `:id` attribute for your model. Any attributes specified in the attributes line will be included in the JSON object that represents instances of your model. To add additional attributes, just add the attribute names as symbols after `attributes`. Using the example of a Post model, this may look like: 

``` 
class PostSerializer < ActiveModel::Serializer
  attributes :id, :title, :body
end
```

You can also delete the id attribute if you so desire (though I like to use ids for data attributes to pass information about my instance to JS click events). 

To represent relationships to other models in your serializer, generate another serializer for that model using the process above, and add the appropriate `has_many`/`belongs_to` relationships to the serializer. Continuing the Post example, if it belongs to an Author, you may write:


`rails g serializer author`

``` 
# app/serializers/author_serializer.rb

class AuthorSerializer < ActiveModel::Serializer
  attributes :id, :name
  
   has_many :posts  
end

# app/serializers/post_serializer.rb
class PostSerializer < ActiveModel::Serializer
  attributes :id, :title, :body
  
  belongs_to :author 
end
```

**NOTE:** Serializers can also be generated with the resource generator if the Active Model Serializers gem is installed when the resource is created. If this is the case, basic attributes and relationships may also be included in your generated serializer. 

**NOTE 2:** Active Model Serializers is undergoing renovations. The [jbuilder gem](https://github.com/rails/jbuilder) is an alternative that works out of the box when running a scaffold generator. 

### Edit controller to render json

There are two different ways (that I know of) to tell the browser to render JSON in your controller. Using the example of  a `posts#index` action: 

```
def index
  @posts = Post.all 
  render json: @posts 
end 
```

and 

```
def index
  @posts = Post.all
  respond_to do |format|
    format.html
    format.json {render json: @posts}
  end 
end 
``` 

Let's do a little experiment. Set the body of your model's index action to the first option, and fire up your Rails server. Visit the index action in your browser, using something like [localhost:3000/posts](http://localhost:3000/posts). Presuming you have an `index.html.erb` file in the `views/your_model` directory, your beautiful index template should be replaced in the window with bare JSON.

The first option changes your action to exclusively render JSON. The controller will ignore any HTML it may otherwise assume you want to be rendered on the page.

Now, set the body of your index action to the second option and refresh the page. You should see your pre-existing index template render in the browser. Add “.json” to the end of your URL ([localhost:3000/posts.json](http://localhost:3000/posts.json)), and you should see the bare JSON rendered in your browser again. Add “.html” to the end of your URL, and you’ll see the index.html.erb template rendered in your browser.

Why does this happen? Using `respond_to` tells your browser, “If the user wants to see the content of this URL as HTML (the default), I'll serve them the content as HTML. If the user requests the data in JSON (data being the information from the database saved in @posts), just add .json at the end of the URL and I’ll show it to them that way.” 

Adding the ability for your controller actions to render JSON enables the Javascript you write to access information from your database in a format it natively understands.

### Now you’re all set up to start writing JavaScript!

My next blog post will be about commandeering the DOM with JavaScript event listeners to paint your browser with content retrieved and sent using asynchronous requests to your database.
