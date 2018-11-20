---
layout: post
title:      "React/Redux Final Project"
date:       2018-11-16 01:59:54 -0500
permalink:  react_redux_final_project
---

ACTIVE STORAGE + REDUX —> IT IS POSSIBLE.

This is my attempt to piece together advice across the Internet* to write a tutorial for using ActiveStorage with React. Follow these steps to upload any file, be it mp3, jpeg or pdf, from a form on a React/Redux application to Active Storage on a Rails API. 

*This post would not have been possible without the amazing help of Jessie Huff, Dakota Martinez, and the gracious souls who responded to numerous Github Issues and StackOverflow Questions.*

This tutorial uses a local storage system and does not use direct uploads. I hope to write addendums to this post to accommodate those processes as I understand them better.

There are tons of awesome tutorials that go in depth on setting up a React App with a Rails API on the backend. I got started using the guides by [Full Stack React](https://www.fullstackreact.com/articles/how-to-get-create-react-app-to-work-with-your-rails-api/) and [Nick Hartunian](https://medium.com/@nick.hartunian/rails-5-api-create-react-app-full-stack-heaven-2b2160b5ce6b). 

After starting your basic React App with a Rails API, we’ll:
Install ActiveStorage
Set up a model, controller, and serializer to handle file attachments
Create stateful React components connected to the Redux store to upload and display your content
Generate reducers and actions to make asynchronous requests to your Rails API


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

Now you have the basic skeleton for your app as well as a command to start both your Rails API (located at localhost:3001) and your React app (located at localhost:3000) simultaneously. Just type: 

```
$ rake start
```

Beautiful! You should see the spinning React logo open in a browser window. If you navigate to localhost:3001, you should be greeted by our Rails cartoon friends.

Now for the fun stuff:  
### Install active_model_serializers gem

This gem prepares Model attributes to be rendered into JSON. Down the line, we’ll use it to include the url for our attached file in the JSON representation of our model. Add it to your Gemfile and run bundle install.

```
# Gemfile

gem ‘active_model_serializers’
```
```
$ bundle install 
```

NOTE: Active Model Serializers is, at the time of writing, undergoing renovations. Rails may have other approved methods/processes in the future.   ### Create the model you’d like to attach a file to

For the sake of this tutorial, we’ll run a scaffold generator for our model. This will create a controller with actions ready to render JSON (thank you API mode!), a model, a serializer with attributes pre-filled, and a migration ready to run for our DB. 

``` $rails g scaffold posts title:string body:string
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
$ rails db:migrate
```
This will generate two tables in your application’s database, `active_storage_blobs` and `active_storage_attachments`. Previous solutions required columns to be added to existing models to accommodate attachments. 

Instead, Attachment is a join model that connects Blobs (which stands for Binary Large OBject) to your models. 

According to [Evil Martians](https://evilmartians.com/chronicles/rails-5-2-active-storage-and-beyond), `active_storage_blobs` don’t put the binary into your database, but tracks the location of the binary file and its associated metadata. 

### Associate Model, Controller, and Serializer with File

Model: 

To associate a file with your model, you just need to add `has_one_attached` and then the attribute name for that file to your model. The attribute name can be anything you’d like.

```
# app/models/post.rb

class Post < ApplicationRecord

has_one_attached :file

end
```
If you’d like to associate multiple files with an Active Record Model, you can use `has_many_attached` instead. I haven’t tested the rest this tutorial using the `has_many_attached` association.

Controller: 
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

Serializer: 

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

  attributes :id, :title, :body, :file_url

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

### YOUR RAILS API IS OFFICIALLY READY FOR LIFTOFF ### 

Take a brief intermission before we dive into the React portion. Perhaps, by watching this lovely video: 


<iframe src="https://player.vimeo.com/video/27315673" width="640" height="360" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
<p><a href="https://vimeo.com/27315673">Trim</a> from <a href="https://vimeo.com/peteyboy">Peter Simon (Petey Boy)</a> on <a href=“https://vimeo.com">Vimeo</a>.</p>



Okay, now we’re back. It’s React time.

For brevity’s sake, we’re going to just use the pre-made App component for our own devices. To truly follow React’s presentational/container pattern and take advantage of the beauty of components for a single purpose, I would recommend creating separate components for: 
the form that holds the upload field
the container that displays the content from the api
the individual records retrieved from the api

If you haven’t already, check out [Thinking in React](https://reactjs.org/docs/thinking-in-react.html) to get up to speed on the process. Long story short (but hopefully not made longer by this intermission), this tutorial is skipping best practices and encouraged design patterns to get to what’s necessary to make Active Storage uploads happen. 

At this point, you’ve gone through the hard and cryptic stuff. The rest is just building a React application with a Redux store that uses Thunk middleware to make POST and GET requests to your Rails API. 

### Prepare your React application to use Redux and Redux-Thunk

Redux is a state management tool that works with React to have one consistent state object, known as the store, accessible to all connected components. This makes the process of accessing passing props between components without direct relationships a lot easier. 

The store operates as a single source of truth for the Redux application, allowing data to be accessed more quickly. 

Instead of making database calls every time a component is rendered, the store holds data related to the current state of your application and passes that data to the components that need it. 

The store updates through actions (Javascript Objects with a key of “type”) and reducers (switch/case statements that alter the state based on the actions dispatched to them). 

Thunk is a middleware for Redux that makes life a lot easier to make asynchronous requests. 

Redux has a built-in function called dispatch that passes actions (which are just plain ol’ JavaScript objects with a key of “type”) down to reducers. According to [the docs](https://github.com/reduxjs/redux-thunk), “a thunk is a function that wraps an expression to delay its evaluation.” Calls to external sources, are asynchronous. Because 

To break it down: 
Redux is not automatically part of React, it needs to be installed
React passes props down from parent components to child components, making it difficult for cousins to get access to that data 
Redux creates a store that is a single source of truth for the application’s current state. 
The store can be accessed by any component connected to it
Redux uses actions and reducers to dispatch changes to the store 

Gaining these powers is as simple as running:

```
$ cd client
$ npm install --save redux
$ npm install --save react-redux
$ npm install —save redux-thunk
$ touch src/reducer.js
``` 
Your React app now has the ability to: 

Hold a store that functions as a single source of truth for the state of the application ([Redux](https://redux.js.org/)) 
Dispatch actions from components to alter the store and read data from the store ([React-Redux](https://react-redux.js.org/docs/introduction/quick-start))
Write action creators that return functions instead of actions allowing asynchronous requests ([Thunk](https://github.com/reduxjs/redux-thunk))

The final command created a file to store our future reducer in, the place where dispatch will send its commands.

There’s one more thing to add before we get started. Our Rails API is ready to accept asynchronous requests from our React application, but our React application does not know where to find our Rails API. Head over to `client/package.json`. 

Add the following key/value pair to the first object, right above the key of `“dependencies"`: 

```
“proxy”: “http://localhost:3001",

```
Instead of writing the entire API URL every time we make a fetch request, now our React app will automatically prefix the path to include the proxy. 

Great! Let’s put these new powers to use! 

### Set up index.js to handle middleware and provide the store 
Add the following into your index.js

```
// client/src/index.js 

import { Provider } from ‘react-redux’;

import { createStore, applyMiddleware, compose } from ‘redux’;

import thunk from ‘redux-thunk’; 

import reducer from ‘./reducer 

```
Provider is a component that connects the Redux store to the React app. It passes down the store as a prop. Provider is the parent component to App — the top-level component for our React application. As a child, App also receives access to the store.

Next, we import three key Redux functions: `createStore` initializes the store based on a reducer and has a second argument containing middleware, which is created by calling `applyMiddleware`. For our purposes, `applyMiddleware`’s argument will be `thunk`. If you’d like to use the Redux DevTools Extension, `compose` allows multiple pieces of middleware to be added to the store upon initialization. 

We put these into action after the import statements with the following: 

```
// client/src/index.js

…

const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ || compose

let store = createStore(reducer, composeEnhancers(applyMiddleware(thunk)));

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root'),
);
```

The first part, `composeEnhancers` connects our application to Redux DevTools, providing a view of dispatched actions and the store’s current state in the browser’s console.

Next, the store is created by calling the `createStore` function with two arguments: the `rootReducer`, which we’ll create in a moment, that contains all of the case/switch statements that will manipulate the store, and middleware connections. Since we’d like to access both the Redux DevTools and Thunk, we use `composeEnhancers` with `applyMiddleware(thunk) `as its argument. If you don’t want to use DevTools, you can also just pass `applyMiddleware(thunk)` as the second argument.

###  Build a stateful component with a file upload field

Let’s create a component to hold our upload form.

```
$ touch client/src/FormContainer.js 
``` 

Create a React component called FormContainer, and connect it to the Redux store. 

```
// client/src/FormContainer.js 

import React, { Component } from ‘react’
import { connect } from ‘react-redux’

class FormContainer extends Component {

  render(){
    return(
      <div>
        <h2>Upload File</h2>
        <form>
          <input type=“text” name=“title” id=“title” placeholder=“title” />
          <input type=“text” name=“body” id=“body” placeholder=“body” />
          <input type=“file” name=“file” id=“file” />
          <input type=“submit” />
        </form>
     </div>
    )
  }
}

export default connect()(FormContainer)
 
```
And while we’re at it, let’s import the FormContainer to our App component, our topmost component, to check our work as we go. 

```
// client/src/App.js



```

Right now, our `FormContainer` component will render HTML to create a form with a title, body and file upload field. The final line connects the component to the store, but doesn’t yet have access to any props or actions from the store. If you submit the form at this point, the information would go nowhere. We need to hijack the `onSubmit` action for the form and the `onChange` actions for the input fields to prepare our data to send to an action. 

To do this we’ll: 
Give the form a local state object that contains keys for each of the file fields

```
// client/src/FormContainer.js 

import React, { Component } from ‘react'
import { connect } from ‘react-redux’

class FormContainer extends Component {
  constructor(props){
    super(props)
    this.state = {
      title: '',
      body: '',
      file: null
    }
… 

```

Bind handleOnChange and handleOnSubmit functions to `this`, giving the functions access to the component’s state 

```
// client/src/FormContainer.js 

import React, { Component } from ‘react'
import { connect } from ‘react-redux’

class FormContainer extends Component {
  constructor(props){
    super(props)
    this.state = {
      title: '',
      body: '',
      file: null
    }
    this.handleOnChange = this.handleOnChange.bind(this)
    this.handleOnSubmit = this.handleOnSubmit.bind(this)
  }

… 

```

Add onChange and onSubmit listeners for each of the fields

```
// client/src/FormContainer.js 

import React, { Component } from ‘react'
import { connect } from ‘react-redux’

class FormContainer extends Component {
  constructor(props){
    super(props)
    this.state={
      title: '',
      body: '',
      file: null
    }
    this.handleOnChange = this.handleOnChange.bind(this)
    this.handleOnSubmit = this.handleOnSubmit.bind(this)
  }

  render(){
    return(
      <div>
        <h2>Upload File</h2>
        <form onSubmit={this.handleOnSubmit}>
          <input type="text" name="title" id="title" placeholder="title" onChange={this.handleOnChange} />
          <input type="text" name="body" id="body" placeholder="body" onChange={this.handleOnChange} />
          <input type="file" name="file" id="file" onChange={this.handleUpload} />
          <input type="submit" />
        </form>
     </div>
    )
  }
}

export default connect()(FormContainer)
… 

Notice the file field is pointing to a different event handler for its onChange property. For text fields, such as title and body, we can use a common handleOnChange pattern, setting the state based on the event target’s name and value: 

```
  handleOnChange = event => {
    this.setState({
      [event.target.name]: event.target.value
    })
  }

```
To have the state always reflect the current value of the input field, let’s set the value in each text input field to the name of the field: 

```
<input type="text" name="title" id="title" placeholder="title" onChange={this.handleOnChange} value={this.state.title} />

<input type="text" name="body" id="body" placeholder="body" onChange={this.handleOnChange} value={this.state.body} />

```

For the file field, instead of setting the state to the value of the event target, we need to set it based on first item in the files property array

```
  handleUpload = event => {
    this.setState({
      file: event.target.files[0]
    })
  }

```

This puts all of the file’s important information and metadata in the component’s state, ready to pass to the onSubmit function, and furthermore our dispatched action.

For `handleOnSubmit`, the function starts out as most submit functions do for regular text inputs:

```
handleOnSubmit = event => {
    event.preventDefault()
    const title = this.state.title
    const body = this.state.body
    const file = this.state.file
    const post = {post: {title: title, body: body, file: file}}

```

This prevents the form from submitting in its standard fashion, pulls the current state of each input field (set through the `handleOnChange` and `handleUpload` functions), and combines those values into a nested object that reflects the format our Posts controller expects, with the name of the model on the outer level, and the attributes on the inner level.

And finally, we close by resetting the form to its empty state:

```
this.setState({
  title: ‘’,
  body: ‘’,
  file: null
})
document.getElementById(“file”).value = null

```
Though `setState` makes the state version of the file null, we also need to use `document.getElementById` to reset the value of the file field so the previous file’s name is no longer present next to the upload button.

### Create an action to make a post request to your API 

Currently `handleOnSubmit` function isn’t sending our data anywhere. Ultimately we want to make a `fetch()` request that POSTs the data to our Rails API. To do this, we need to create an action.

```
$ touch src/actions.js 
```

In the `actions.js` file, we’ll use thunk to make our post request. 


Dispatch an action letting the store know that we’re taking an asynchronous action

```
export function addPost(post)
  return (dispatch) => {
    dispatch({ type: ‘START_ADD_POST_REQUEST’ })

```

Convert the data from our component into a format that’s friendly to both JSON and our Rails API using the built-in JavaScript FormData object and appending our data to it

```
      const postData = new FormData()
      postData.append("post[title]", post.post.title)
      postData.append("post[body]", post.post.body)
      postData.append("post[file]", post.post.file)

``` 
Make a fetch request to POST to the Rails API with our `postData` as the body and convert the response to JSON

```
      return fetch('/api/sounds', {
        method: 'POST',
        body: soundData,
        contentType: false,
      })
      .then(resp => resp.json())

```
Dispatch the JSON version of the response to your reducer

```
.then(post => dispatch({ type: ‘ADD_POST’, post }))

```

The whole function should look something like this: 

```
// client/src/actions.js

export function addPost(post){
  return (dispatch) => {
    dispatch({ type: 'START_ADD_POST_REQUEST' })
    const postData = new FormData()
    postData.append("post[title]", post.post.title)
    postData.append("post[body]", post.post.body)
    postData.append("post[file]", post.post.file)
    return fetch('/posts', {
      method: 'POST',
      body: postData,
      contentType: false,
    })
    .then(resp => resp.json())
    .then(post => dispatch({ type: 'ADD_POST', post }))
  }
}


```

Our reducer will receive the final action, `{type: ‘ADD_POST’, post}`. We need to create a reducer that holds an initial state for our Post model, responds to the `ADD_POST` action type and adds our post to the store. 

Create and export your reducer function. The first argument should be what your initial state will look like, in this case, an object with an array of posts. The second argument is action, which fill be passed with whatever action object dispatch sends to the reducer. 

```
// client/src/reducer.js 

export default function reducer(state = {posts: []}, action){
  
}


```
Write a switch statement with an argument of action.type, and add the case for our ‘ADD_POST’ action and a default response that returns the current state. 

```
// client/src/reducer.js 

export default function reducer(state = {posts: []}, action){
  switch(action.type){
    case 'ADD_POST':
      return [...state, action.post]
      
    default:
      return state;
  }
}

```

The `ADD_POST` case statement’s return value will concatenate the information from the fetch request to the application’s store. So adding a new post should make the state look something like this: 
>>>>>>>>>>>>>> ADDD EXAMPLEEEEE >>>>>>>>>>>>>>>>>


Now that our action exists, include it in our connect function within the `FormContainer`. 

First, import the action into the document 

```
import { addPost } from './actions.js'

```

Within the call to the connect function in the export statement for `FormContainer`, add two arguments

``` 
export default connect(null, { addPost })(FormContainer)

```

null is the place reserved for mapStateToProps, which passes down information in the store for components to use. `{ addPost }` is in the place of mapDispatchToProps. The curly braces in this case take place of explicitly calling the dispatch function (`dispatch{ addPost }`). By adding this action to the connect function, we can now call it in `handleOnSubmit` and pass our `post` object to it as an argument. 

Within `handleOnSubmit`, between the `const post` declaration and call to `this.setState`, add: 

```
this.props.addPost(post)
```

The entire function should now look like: 

```
  handleOnSubmit = event => {
    event.preventDefault()
    const title = this.state.title
    const body = this.state.body
    const file = this.state.file
    const post = {post: {title: title, body: body, file: file}}
    this.props.addPost(post)
    this.setState({
      title: '',
      body: '',
      file: null
    })
    document.getElementById("file").value = null
  }

```

Now all the functionality is present to render a form to upload a file and send the submitted form’s data as a POST request to your Rails API! Fire up the server, open your Redux DevTools, and let’s make a post! 

You should see something like this in your window: 

(ADD IMAGE) 

Click on the “State” button on the right side of your Redux DevTools Console (“Diff” is automatically selected upon launch). Let’s input some information into our field and see what happens: 

(ADD IMAGE) 

On the right side of your Redux DevTools, you’ll see a list of all the actions dispatched. First, our ‘START_ADD_POST_REQUEST’ was sent, which told the store what was happening in the application. Then, the promise from the fetch request, attached to our ‘ADD_POST’ action, was returned and a new object was added to the posts object in the state.

Let’s peek at our API. Navigate to localhost:3001/sounds.

You should see the JSON version of the object we just posted with the title, body, and file_url attributes. Click on the file_url link, and see your file in your browser! 

It’s all well and good to know our POST requests are working, but what if we want to render this file in the browser? 

All it takes is creating a component to render the items stored in your Rails API, writing an action to submit a GET request, and calling that action in your topmost component (in this case, our App component) during the `componentDidMount` lifecycle method to push the API’s data to the store. 

Phew! Let’s break that down: 

### Create a component to render items stored in your Rails API

```
$ touch client/src/Posts.js 
``

And in that file, write: 

```
import React from 'react'

const Posts = (props) => {
  return(
    <div className="posts">
    </div>
  )
}

export default Posts

```

### Write an action to handle a GET request for the API content 

```
#client/src/actions.js

export function getPosts(post){
  return (dispatch) => {
    dispatch({ type: ‘START_GET_POSTS_REQUEST’ })
     return fetch('/posts')
    .then(resp => resp.json())
    .then(posts => dispatch({ type: 'GET_POSTS', posts }))
  }
}

```

### Write a case statement to handle that action in the reducer

```
# client/src/reducer.js 

… 

    case 'GET_POSTS':
      return {...state, posts: action.posts }

```

### Import the `getPosts` action, the `connect` function, and the Posts component into the App component 

```
import { getPosts } from './actions.js'
import { connect } from 'react-redux'
import Posts from './Posts.js'

```

### Pass `getPosts` to the `connect` function as the argument for `mapDispatchToProps` 

```
export default connect(null, { getPosts })(App)

```

### Write a mapStateToProps function to access the posts object from the store, outside of the component and pass the function as the first argument of the connect() function

```
function mapStateToProps(state){
  return {
    posts: state.posts
  }
}

export default connect(mapStateToProps, { getPosts })(App)

```

### Call getPosts within the componentDidMount() lifecycle method within the App component

By calling the getPosts method during componentDidMount lifecycle method of the App component, the information will be fetched from the database only when the entire Application is reloaded. Any new posts added without the App reload, will be pushed to the store through the ADD_POST action.

```
class App extends Component {

  componentDidMount(){
    this.props.getPosts()
  }

  render() {
    return (
      <div className="App">
        <FormContainer />
      </div>
    );
  }
}


```


### Add the Posts component return statement below the `FormContainer`, and pass down the posts returned from mapStateToProps as a prop.

```
class App extends Component {

  componentDidMount(){
    this.props.getPosts()
  }

  render() {
    return (
      <div className="App">
        <FormContainer />
        <Posts posts={this.props.posts} />
      </div>
    );
  }
}


```

### Use the posts props to render individual posts on the page 

Returning to our Posts.js file, iterate through the post objects passed down from the App component, and render each object as an <li>.

```
import React from 'react'

const Posts = (props) => {

  const renderPosts = this.props.posts.map(post => {
    <li key={post.id}><strong>{post.title}</strong> - {post.body} - {post.file_url}</li>
  })

  return(
    <div className="posts">
    {this.renderPosts}
    </div>
  )
}

export default Posts


```

There you have it! Thanks for reading! 

