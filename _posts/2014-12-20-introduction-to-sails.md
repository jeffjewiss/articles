---
title: Introduction to Sails.js
date: 2014-12-20 00:00:00 -05:00
layout: post
---

Sails is a [framework](http://sailsjs.org/) and a set of tools to help you build well structured Node.js applications, prototypes and APIs. It provides built in support for web sockets, MVC structured code and a CLI for quickly scaffolding your API, while building everything as an interchangeable node module.

Sails acts as both a command line interface, for setting up new projects and generating files for your application, and the server library for running your application. The toolset is installed globally, while the server is just another dependency of your application.

## The Benefits of Sails

Convention over configuration is one of the main selling points of Rails, and that concept of structure is applied to the Sails way of developing applications in Node. This is a huge benefit for teams building a large number of applications with a varying number of developers working on each project. Sails provides the consistency in building an API or application that made Rails so popular and a boon to productivity.

These features are also highlighted on the Sails website:

* 100% JavaScript
* Any database
* Powerful associations
* Auto-generate REST APIs
* Easy WebSocket Support
* Reusable security policies
* Front-end agnostic
* Flexible asset pipeline
* Rock-solid foundation


## Getting Setup

The first thing you'll need to do is install the command line tools by globally installing the Sails package:

```bash
npm install -g sails
```

Once this finishes successfully you can create a new sails project (and a new directory) by running:

```bash
sails new <project-name>
```

This will start you off with all the files you need for your application, as well as the basic set of dependencies. It's important to note that as of Sails version 0.10.5, the dependencies in your node modules directory will be symlinks to the global versions already installed. If you'd prefer to use local copies, you can delete the node modules directory and run `npm install`.

To start the server and get your application started, switch to the directory where the application was created and run:

```bash
sails lift
```

You'll see log information on the actions Sails is taking and the URL where you can view the running application. To quickly open the URL in your browser, you can <ctrl> + click the link, which is likely `http://localhost:1337`.


## Application Internals Overview

So what's in a Sails application and what's happening when you run `sails lift`?

Sails is running an Express server with Connect compatible middleware as well as Waterline, an ORM for handling connections to your database and operations on your data through the use of models. Sails also sets up web sockets for you, handles routing through a configuration module and controls access to data and actions through its policy middleware and compiles all of your views, javascript and stylesheets for you.

All of these pieces are organized in files and directories familiar to those working in other MVC frameworks, with server side code in an `api` directory, settings in a `config` directory, and front end code in an `assets` directory, which is compiled using Grunt by default. These pieces are all modules, which can be swapped out and more modules can be added in a similar way. If you don't like Grunt, you can easily swap it out for Gulp or something else. If you want to use the Passport module for authentication you can get up and running by installing the module and creating the appropriate configuration for its middleware.

Sails is a collection of conventions and modules to help you build Node web applications quickly in a flexible manner and within a framework that allows you to tailor it to your liking by swapping out any of the internals.

## Creating Some Views

After running `sails new` to create a new Sails application you should have a homepage view, a layout and some error pages, but not much else. If you open up the router configuration file at `config/routes.js` you should now have one route for the root URL of your application:

```javascript
'/': {
    view: 'homepage'
}
```

So if you wanted to add an about page to your application you would add a new route so that the above code would become:

```javascript
'/': {
    view: 'homepage'
},
'/about': {
    view: 'about'
}
```

Now when you visit `http://localhost:1337/about` in your browser Sails will look for a view named "about" to render, after wrapping it in the layout template. This view does not exist yet, so we'll need to create a new file `views/about.ejs` and populate it with some HTML. Since these views don't currently have a controller no data from the API will be available to use in the view, but you will have access to localized text, view variables and other data provided by the application's middleware.


## Using the Blueprint API

The Blueprint API is a set of default routes and actions that provide instructions for your application to handle the basic CRUD (create, read, update, destory) actions.  The internal logic of the Blueprint API powers the RESTful JSON API you get for free whenever you create a new model and controller. In practice, this means that by simply having the default configuration in place and creating an empty model and controller the Blueprint API will provide URLs for you to create a new instance of that object, see an array of all the instances and so on (CRUD operations). Also, Sails provides generators to create models and controllers, so you can accomplish this without writing a single line of code. This is incredibly powerful for prototyping and getting an API together very quickly.


### Example Usage

So if you wanted your API to have a book object, you could generate a model and controller for it using the Sails CLI by running:

```bash
sails generate api book
```

*If Sails interrupts and asks for a migration setting, you can type 2 and press enter.*

Now you could create a new book in your application by simply visiting the URL `/book/create?name=Game of Thrones`, and another one by visiting `/book/create?name=The Hunger Games` and then view a list of your books by visiting `/book`.

*note: to avoid the migration warning from Sails when generating the book api, add the following to `config/env/development.js`*

```javascript
models: {
  migrate: "alter"
}
```


### Blueprint Routes

When Blueprints are enabled (which is the default) and you run `sails lift`, the framework inspects your models and controllers to create "shadow" routes to respond to many of the common requests without you having to configure them in `config/routes.js` or otherwise write any code. These "shadows" will respond to these requests by pointing to the corresponding Blueprint actions.

There are 3 types of Blueprint routes in Sails:

- restful routes
- shortcut routes
- action routes

and each type can be configured independently.


#### Restful Routes

These routes always have the path of `/:modelName` or `/:modelName/:id` and send the request to the appropriate action by using the HTTP "verb". Middleware policies should be used in a production environment to protect these routes from unauthorized access.


#### Shortcut Routes

These routes only respond to "get" requests and determine which action to send the request to by decoding the path. An example path would look like `/:modelName/<action>` and data would be passed to the controller action using query parameters. While great for development work on a prototype, these routes should be disabled in production.


#### Action Routes

These routes create shortcut routes for custom actions that don't come for free as part of the restful routes. So for any custom action on a controller, a corresponding path following the format `/:controllerName/:actionName` will respond to get requests and send the request to the controller.


### Blueprint Actions

The Blueprint API creates a number of generic actions to handle all of the standard behaviour of a Restful JSON API to match the Blueprint routes. The following default controller actions, which can be overridden, are provided by the Blueprint API:

- find
- findOne
- create
- update
- destroy
- populate
- add
- remove


## Configuration Basics

The instructions that define how your application will run live in the "config" folder as a set of modules. These modules come with a set of a sensible defaults and conventions that will allow you to get up and running with a prototype very quickly, and you likely won't need to configure anything if you just want to setup an API to handle model data or create some server pages using views. Once you've moved past the prototype stage and are beginning to build out your application, it's useful to understand how to perform some common configuration tasks.

### Routing

While basic routing has already been covered, it's important to understand how to use the HTTP verbs and dynamic paths in your routes for when you start to build out your API. A couple of examples will help illustrate how to use both of these features.

If you wanted to setup routing for an API endpoint that responded to a get request with a list of all the posts, you would have:

```javascript
'get /posts': {
    controller: 'postsController',
    action: 'list'
}
```

This configuration would tell the application to respond to "get" requests at the specified URI and send that request to be handled by the "list" action of the "postsController".

If you wanted to setup routing for an API endpoint that responded to a "put" request to update an existing user, you would have:

```javascript
'put /posts/:id': {
    controller: 'postsController',
    action: 'update'
}
```

This configuration would tell the application to respond to "put" requests at the specified URI and pass the request, including the id parameter, to the "update" action of the "postsController".


### Using Jade Instead of EJS

To swap out a different view engine you'll need to follow a few steps. The first thing you'll need to do is install the node module for the engine you'd like to use and add it to your `package.json` file.


```bash
npm install jade --save
```

Next you'll need to update the configuration file for you views in `config/views.js`:

```javascript
module.exports.views = {
    engine: 'jade',
    layout: false,
    locals: {
        // Any options you would like to pass to the Jade parser
    }
}
```

You'll need to disable the layout option, since this is only currently supported for EJS, and instead use the engine's extend functionality (a combination of extends and blocks in Jade).

Finally you'll want to remove the EJS module from your `package.json`.


### Using Sass Instead of Less

This can be achieved by simply swapping the Grunt tasks. First remove the `grunt-contrib-less` task from your `package.json` and install the Grunt Sass module:

```bash
npm install grunt-contrib-sass --save
```

Next change all Grunt task references of "less" to "sass". Start by renaming the file `tasks/config/less.js` to `tasks/config/sass.js`. Also, change all of the references of "less" to "sass" within the file, as well as in the following files:

* "tasks/config/copy.js"
* "tasks/register/compileAssets.js"
* "tasks/register/syncAssets.js"

And finally you'll need to change the extension on any `.less` files to `.scss` and update any of the styles themselves that do not follow the Sass syntax. If you just created a new project you'll only need to change `importer.less` and all of the contents should be compatible with Sass.

### Using Postgres Instead of LocalDB

As briefly mentioned before, Sails uses an Object Relational Mapper, called Waterline, to provide a consistent interface when dealing with model data in your application. This ORM works with  many popular databases, like Postgresql, MongoDB and Redis. Waterline has an accompanying adapter, which is a node module, for each database you'd like to use. New Sails apps start of by using the "sails-disk" adapter, which creates and modifies a javascript file with all of the application's data on the filesystem where Sails is running. This is great for getting up and running quickly and prototyping, but wouldn't do for a real app.

You've probably noticed a pattern by now. The first step is to install the Sails Waterline adapter for Postgresql:

```bash
npm install sails-postgresql --save
```

Once installed, you'll need to configure the adapter in the `config/connections.js` file by modifying one of the examples or adding a new connection. The following example configuration uses a URL via an environment variable instead of a username, password, host, and port because it's best practice to keep your API keys and usernames and passwords out of your code, and deployment will be done using Heroku.

```javascript
postgres: {
    adapter: 'sails-postgresql',
    url: process.env.DATABASE_URL,
    pool: true
}
```

With this configuration in place various parts of your application can now connect to and use your Postgres database, however nothing has been configured to use it. Also, it's worth noting that adding another type of database would follow the same pattern of installing the adapter and configuring the connection.

Now, to setup your models to use the Postgres adapter the `config/models.js` file will need to be updated:

```javascript
module.exports.models = {
    connection: 'postgres'
}
```

#### Environment Variable Setup

To setup your local copy of the application to use a "database url" variable first create a file at the root of your project with the name `.env` and add the line `DATABASE_URL=postgres:///jeff`, but instead of "jeff" put the username you use to login to your computer. This assumes that you already have Postgres installed and it will use the default settings to connect. It's considered best practice to add `.env` to your `.gitignore` file so that it is never committed to your git repository since different environment variables will be provided when you deploy the application.

You'll need to install a tool to load the "database url" variable into the environment when you run Sails. Foreman is the standard tool for loading environment variables, and a Node version can be installed by running `npm install -g foreman.`

Foreman will check for a start script to know how to run your application. In the case of a Sails application, this is equivalent to `sails lift`. Add a start script to your `package.json`:

```javascript
scripts: {
    "start": "node app.js"
}
```

You can now run the application locally using `nf start` instead of `sails lift`.


## Deploying to Heroku

Once the application has matured to a point where you want to share it with the world, or maybe just a few friends, it's time to deploy it. Deploying a Sails application to Heroku is fairly straightforward and best of all it's free, assuming you use the default settings and one dyno.

*Note: if you haven't already created a git repository for you application, you will need to do so to deploy an application to Heroku. This can be done by running `git init`.*

The first step is to install the [Heroku Toolbelt](https://toolbelt.heroku.com/) and login, or create an account if you don't already have one. Now you'll be able to create a copy of your application on Heroku:

```bash
heroku create
```


### Configuration

At this point in the development of our application there isn't much to configure to be able to deploy it. However, we still need to ensure that Heroku will be able to start and run the application and handle any database connections.

Since a start script has already been added, Heroku already knows how to run the application. However, a Postgresql database still needs to be added:

```bash
heroku addons:add heroku-postgresql:hobby-dev
```

This command adds the free tier Postgres database to your Heroku application and also adds the `DATABASE_URL` environment variable for you.

To ensure that migrations are handled automatically by Sails after each deploy, you'll want to update the production model configuration in `config/env/production.js`

```javascript
models: {
  migrate: "alter"
}
```

*Note: for a real production application this setting should be `migrate: "safe"`, which requires you to manually handle migrations.*


### Deploying a Version of the Application

Heroku handles transferring code and building applications with Git. When you ran the `heroku create` command a remote named "heroku" was added to your local git repository. To send the code to Heroku and have it build and run your application you just push to the "heroku" remote:


```bash
git push heroku master
```


## Summary

Let's be honest, your application doesn't do much. At this point you likely have the default Sails homepage, a sparse about page and maybe some models and controllers powered by the Blueprint API. However, it's been deployed to Heroku so you can share it with your friends, and have a basic understanding of how Sails works and what it can do for you.

From here you should be able to create a website with some static pages served by Sails or a basic API for development purposes. It may not seem like much to show off, but it's a solid foundation to build on.
