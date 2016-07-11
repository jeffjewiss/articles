---
title: Quick and Free Static Sites with Metalsmith and Divshot
date: 2015-02-25 00:00:00 -05:00
layout: post
---

Sometimes you just want to be able to build something with your favourite tools and get it online quickly. There are a lot of options available today, but I’ve settled on a few favourites, which I’ve turned into a repeatable process for getting projects live quite quickly.


## Background

Let’s start with a couple definitions on what I’ll be covering. There are two main components: the build step, and hosting the files. The former involves all of the programs and tooling to take the website files as authored in a preferred language and convert them into HTML, CSS and JavaScript that can be served up to a browser as static assets. The second, the hosting provider, is the service that will serve these files and the process involved in getting the files to the provider and the options available to manage the site.


### Building the Site with Metalsmith

[Metalsmith](http://metalsmith.io) is a static site generator written in JavaScript that is fast, flexible and easy to use because of its plug-in structure.

The core of Metalsmith works in three steps:

1. Read all the files in a source directory.
2. Invoke a series of plug-ins that manipulate the files.
3. Write the results to a destination directory!


So in practice this means you setup a source directory where you author your website, create a small Metalsmith program with configuration to use your flavour of tooling, and run the program to create the final output to be viewed in the browser, typically in a build directory. Metalsmith will typically process anything in the source directory, which means anything to be excluded from the build (ie. templates, partials, metadata), should be stored outside of your source directory. Don’t worry, Metalsmith can still access these files, they just won’t result in files being created in the build directory.

The Metalsmith collection of plug-ins includes support to use a variety of template languages, CSS preprocessors, Markdown processors, code optimizers and other tools to provide flexibility in how you can use the tool. Later I’ll give an example of how I’m currently using the tool to quickly create static sites.


### Deploying with Divshot

[Divshot](https://divshot.com) is a PaaS (Platform as a Service) similar to Heroku in that it allows you to deploy websites with a CLI (command-line interface), however one of the main differences is that Divshot is only for static websites. This can range from simple HTML and CSS pages to full blown applications using Ember, Angular, React, Ampersand, Browserify modules or however you decide to structure structure your application.

Divshot is also similar to Heroku in that it takes a compiled build of your application and combines it with some configuration to create an immutable release. This means that you can use whichever build process you’d like with Divshot since it only cares about the finished build output. The configuration also includes routing, which means you can have your application respond to allow routes or even have multiple build targets as part of the same deploy as well as supplying error pages.


## The Example

The site I’m using as an example is one that is currently live that uses markdown for content, jade templates and SUITCSS to build a couple of pages. It’s a little overkill for such a small project, but the tooling choices are some of my favourites to use, so feel free to drop in your favourites in their place.

Below is the file structure for the project. I’ll be covering the contents of `build.js`, `package.json`, `Procfile` and how to use local variables and specify a template in one of the Markdown files.

```
▸ build/
▸ node_modules/
▾ public/
    favicon.ico
    robot.txt
▾ src/
    error.md
    index.md
    styles.css
▸ templates/
  .editorconfig
  .gitignore
  build.js
  divshot.json
  package.json
  Procfile
  README.md
```

*Note: it’s a good practice to add "node_modules" and "build" to your `.gitignore`*


### Build Script

First, we’ll go over the `build.js` file, which contains the entirety of the Metalsmith application.

```javascript
var metalsmith = require('metalsmith');
var templates = require('metalsmith-templates');
var markdown = require('metalsmith-markdown');
var suit = require('metalsmith-suitcss');
var fingerprint = require('metalsmith-fingerprint');
var asset = require('metalsmith-assets');
var htmlMinifier = require("metalsmith-html-minifier");
var cleanCSS = require('metalsmith-clean-css');

metalsmith(__dirname)
  .source('src')
  .destination('build')
  .use(markdown({
    gfm: true
  }))
  .use(htmlMinifier())
  .use(suit())
  .use(cleanCSS())
  .use(fingerprint({
    pattern: '*.css'
  }))
  .use(templates({
    engine: 'jade',
    directory: 'templates'
  }))
  .use(asset())
  .build(function (err) {
    if (err) throw err;
  });
```

The entire build script consists of declaring dependencies, creating an instance of Metalsmith and using a number of plug-ins.

When Metalsmith is created using `metalsmith()` and it takes a directory as an argument. This argument is the directory where Metalsmith will run and in this example `__dirname` is used, which refers to the directory where the current program is being executed.

An instance of Metalsmith has a few important methods that make up the bulk of the API for building a site, which are “source”, “destination”, “use” and “build”. Source and destination will, respectively, tell Metalsmith where to find the files it will transform and where to put the compiled website when it’s done.


### Plug-in Usage

I’ll briefly go over each Metalsmith plug-in used in this project. First, it’s important to note that the `.use()` method takes an instance of a plug-in as its single argument and the plug-in itself typically takes a hash of options as its single argument. This creates a common usage pattern, of chained methods, for Metalsmith where options and plug-ins are defined and then the build method is finally called.

*Note: the order in which the plug-ins are executed can matter, which is why the CSS related plug-ins are called before “templates”.*

**markdown**
Parses files with a markdown extension in your source directory and converts them into the corresponding HTML markup.


**htmlMinifier**
Removes whitespace from any HTML files.


**suit**
Uses the SUITCSS preprocessor tool on any CSS files.


**cleanCSS**
Minifies and removes whitespace from any CSS files.


**fingerprint**
Creates a copy of any files specified and adds a unique hash to its file name based on the file contents.


**templates**
Uses the consolidate.js library on specified template files and allows for the usage of layout files to be defined on other source files using YAML.


### Dependencies and Shell Scripts

```javascript
{
  "main": "build.js",
  "scripts": {
    "start": "node ./build.js",
    "deploy": "npm start && divshot deploy"
  },
  "dependencies": {
    "jade": "~1.8.2",
    "metalsmith": "^1.0.1",
    "metalsmith-assets": "^0.1.0",
    "metalsmith-clean-css": "^2.0.0",
    "metalsmith-fingerprint": "^1.0.1",
    "metalsmith-html-minifier": "^1.1.0",
    "metalsmith-markdown": "^0.2.1",
    "metalsmith-suitcss": "^0.1.0",
    "metalsmith-templates": "^0.6.0",
    "normalize.css": "^3.0.2"
  },
  "devDependencies": {
    "browser-sync": "^1.8.3",
    "onchange": "0.0.2"
  }
}
```

The two pieces of note are the “start” and “deploy” properties on the scripts objects. The start script is a node convention for “package.json” files and will execute the build process when you run `npm start`. The deploy script will build the site and then use the Divshot CLI tool to upload the packaged build using the configuration settings found in the “divshot.json” file.


### Process File

To make the authoring process a bit smoother I use a few additional tools to rebuild the site when files change and to inject new CSS into the browser without a refresh. All of these additional tools are optional and you can build and deploy the site without them, but they speed up the feedback loop.

The main tool is the Node version of Foreman, a process manager, which allows you to run several processes at once (each with their own logging) by specifying the custom commands in a process file called “Procfile”. Foreman will create a child process for each line in the file and use any environment variables specified in a “.env” file if you have any API keys or sensitive data you don’t want to commit with git. Here are the contents of a Procfile I’m using:

```bash
metal: npm start && node_modules/.bin/onchange 'templates/*' 'src/*' -- npm start
serve: divshot server build -p 4000 --no-cache
bsync: node_modules/.bin/browser-sync start --files "./build/*.css" --proxy 0.0.0.0:4000 --no-open --no-notify
```

These commands can be executed by running `nf start`.

The metal process uses a package called “onchange” to watch the source files for the site and run the Metalsmith build process any time they change.

The serve process uses the Divshot CLI to serve the static files. This is important because it will honour the “divshot.json” configuration and runs the same server that will be used when you deploy your site.

The bsync process runs [Browser Sync](http://www.browsersync.io/) as a proxy so that any changes to the CSS will be injected into the browser without a page refresh. Browser sync also has other useful features, like syncing scroll position and UI interaction on multiple devices and a remote inspector.


### Templates and Variables

A very quick example of some source files that further explain the workflow are an index Markdown file and a default Jade template. The Markdown file demonstrates how a variable can be defined, in this case the page title, and a template specified using YAML. The Jade file illustrates how you would use a variable, in this case the title, and how you specify where the rendered Markdown would go using the `!= contents` expression.

**src/index.md**

```yaml
---
title: A Page Title
template: default.jade
---

## Subheading

A contrived Markdown example.
```


**templates/default.jade**

```javascript
html
head
  title title
body
  != contents
```


## Conclusion

The above toolchain is serving me well for simple sites that are mostly written material and only a few pages. I found the process of building with Metalsmith very straightforward and was able to create my own plug-in for a workflow that didn’t exist yet ([SUITCSS](https://github.com/jeffjewiss/metalsmith-suitcss)) very quickly. Using Divshot has also been a breeze and I’ve found the performance to be great, even on the free tier.

I think this toolchain will evolve as I continue to use it and could work well with other tools and build targets. It allows you to do quite a bit without having to install any additional applications since it runs entirely on free software.
