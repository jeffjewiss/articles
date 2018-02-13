---
title: Configuring Webpack and Postcss with a Phoenix App
<!-- date: -->
layout: post
tags: [phoenix, elixir, postcss, webpack, babel]
---

##### Versions Used
|
Elixir: 1.5.1
Phoenix: 1.3.0
Webpack: 3.10.0
Postcss Loader: 2.0.10
Babel Loader: 7.1.2

Currently new Phoenix projects come with support for ES6 imports and frontend asset building using [Brunch](http://brunch.io). This works great as a starting point for most projects that have some application level javascript or add a bit of functionality on specific views. Brunch also builds very quickly and the default setup just works if you want to start writing some vanilla javascript or some jQuery to enhance your views.

However, for the application I’m building a few use cases led me to setup and configure Webpack in place of Brunch. These use cases are:

* easily import Turbolinks
* simple Postcss setup
* include fonts from NPM via node_modules

The configuration for Webpack wasn’t straightforward, which wasn’t a surprise, but it is concise once you navigate the various configuration options available to Webpack itself and the various loaders that you will likely leverage.

Remove Brunch
-------------

The first step is to remove all of the Brunch configuration and dependencies from your project. This is currently preferable to running `mix phx.new --no-brunch` for a new project because the `--no-brunch` flag will skip creating the directory and file structure for front-end assets and is better suited to an API only project.


Configure Webpack
-----------------

### Javascript

### CSS with Postcss

### Including Fonts

### Finished Config

Conclusion
----------
