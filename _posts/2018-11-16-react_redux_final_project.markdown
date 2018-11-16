---
layout: post
title:      "React/Redux Final Project"
date:       2018-11-16 06:59:52 +0000
permalink:  react_redux_final_project
---


This is my attempt to piece the advice across the Internet together to share the steps to upload any file, be it it jpeg or pdf, from a form on your React/Redux application to Active Record. This would not have been possible without the amazing help of Jessie Huff, Dakota Martinez, and those who took their time to respond to many, many Github Issues and StackOverflow Answers.

After starting your basic React App with a Rails API, we’ll:
* Install ActiveStorage
* Set up a model, controller, and serializer to handle file attachments
* Create stateful React components connected to the Redux store to upload and display your content
* Generate reducers and actions to make asynchronous requests to your Rails API

There are tons of awesome tutorials that go in depth on setting up a React App with a Rails API on the backend. I got started using the guides by [Full Stack React](https://www.fullstackreact.com/articles/how-to-get-create-react-app-to-work-with-your-rails-api/) and [Nick Hartunian](https://medium.com/@nick.hartunian/rails-5-api-create-react-app-full-stack-heaven-2b2160b5ce6b). 

To fast-forward through the setup:

* Generate a new Rails app in API mode

```
$ rails new app_name --api  
```

* Enter into said Rails API

```
$ cd app_name 
```

* Use create-react-app to make the client directory that will hold your React app

```
$ create-react-app client 
```

To create a rake task that will start both servers at once:  

* Add Foreman to your Gemfile and run bundle install 

```
*# Gemfile* 
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
web: sh -c ‘cd client && npm start’
api: bundle exec rails s -p 3001
```

The first line executes a shell command that first cd's into the client to access the built-in npm tasks and then starts the server at localhost:3000. The second line starts up your API at localhost:3001.

* Create a new rake task to run that command: 

```
$ touch lib/tasks/start.rake 
``` 
* And in that file, paste: 

```
task :start do
  exec 'foreman start -p 3000'
end 
``` 

Now you have the basic skeleton for your app as well as a command to start both your Rails API (located at localhost:3001) and your React app (located at localhost:3000) simultaneously! 

To get your servers running, just type:

```
$ rake start
```

Beautiful! You should see the spinning React logo open in a browser window. If you navigate to localhost:3001, you should see your Rails app.

Now for the fun stuff: 

* install active_model_serializers gem
* create your model
* create your serializer
* scaffold with active model serializers 
* rails g scaffold sounds title:string description:string
* migrate your db 
* install active storage 
* add has_attached_file to model, serializer
* edit controller to contain sound_params if not there already, and add your file attribute to the list of permitted keys
* test using the seed file to see if images are successfully attaching
* write seeds
* run rails db:seed 
* enable rack-cors?
* head over to react app
* create stateful component with file upload field
* create action to handle post request
* create reducer to handle post request 
* connect component to the store 
* create stateful component to render items from db
* create action to handle get request
* create reducer for that action
* call action in component did mount for app (b/c don’t want to fetch all the time)
* add line to config/environments/development for url routes
* update serializer for rails route helpers 
* upload all of the files



