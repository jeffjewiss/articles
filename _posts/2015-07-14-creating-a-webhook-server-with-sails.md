---
layout: post
title: Building a Webhook Server in Sails
---

Webhooks are a simple way to integrate APIs together based on events that occur on one of the APIs. They are also often referred to as HTTP callbacks. This is useful for developers who want to add functionality to their own web application or for connecting third-party services together.

I wanted to setup a quick server to handle webhooks because I help organize an event and I wanted to add attendees to an email newsletter (with double opt-in) when they reserved a ticket. Tito, the service we are using to manage attendance, doesn’t have an API to leverage, but it does have a webhooks feature. While Mailchimp, the service we’re using to manage and send out our newsletter, does have an API. This meant I could create a server that accepts the outgoing webhook from Tito, process the data, and make a request to the Mailchimp API to add the attendee to the newsletter list, which would then send them an email to confirm their subscription.

Webhooks are either incoming or outgoing. For the purposes of the integration described above, the webhook will be outgoing for Tito and incoming for the server I’ll be creating. When a service, like Tito, provides an outgoing webhook feature there is typically a form field where you can provide a URL for the API to make a POST request when certain events occur.


Setting Up the Server
---------------------

To create a new Sails project without a front-end, we’ll follow these steps:

* run `sails new webhooks-example --no-frontend` (replace webhooks-example with anything you’d like)
* disable the grunt hook by adding `"hooks": { "grunt": false }` to the .sailsrc file
* delete the node_modules directory that contains symlinks to globally installed packages
* delete the views directory
* clean up the `package.json` file by removing any references to Grunt
* run `npm install`

Now we have a Sails server without any of the front end, which you should be able to start up by running `npm start`.

The next step is to create the controller and service that we’ll need and install some dependencies. First we’ll generate a controller for handling the webhook from Tito by running `sails generate controller Tito` and then create a file for the service to handle the interaction with the Mailchimp API by running `touch api/services/MailchimpService.js`.

We’ll need to add a Mailchimp API key and list ID, but we’ll want to ensure that these do not get committed to the repo. The recommended way to do this is to use environment variables. Practically speaking that means we’ll create a file called “.env”, which will contain `MAILCHIMP_API_KEY=<key here>` on one line, and `MAILCHIMP_LIST_ID=<list id>` on the next. To use these pairs in our application we can use foreman. We’ll need to install it globally by running `npm install -g foreman` and then use `nf start` to run our application instead of `npm start`.

Lastly we’ll need to install the Mailchimp API package by running `npm install --save mailchimp-api`.


Writing the Service, and Controller Logic
-----------------------------------------

### The Service

Let’s setup the service first and then create a method that handles subscribing an email address to the list.

In “api/services/MailchimpService.js” we’ll add:

```javascript
var listID = process.env.MAILCHIMP_LIST_ID;
var apiKey = process.env.MAILCHIMP_API_KEY;

var mailchimpAPI = require('mailchimp-api');
var mc = new mailchimpAPI.Mailchimp(apiKey);
```

The first two lines grab our environment variables and the second two create an instance of the Mailchimp API wrapper using our API key. Continued in the same file:

```javascript
module.exports = {
  subscribe: function (options, successCallback, errorCallback) {
    if (!options.email) {
      throw new Error('You must provide an email address to subscribe.');
    }

    var requestData = {
      id: listID,
      email: {
        email: options.email
      }
    };

    mc.lists.subscribe(requestData, successCallback, function(error) {
      sails.log.error('There was an error subscribing that user');
      errorCallback(error);
    });

  }
}
```

The idea behind Sails services is to provide a function or object that is globally available to the other parts of your application. It's a way to break off functionality and not overload your models or controllers. In this case the service is pretty simple; it will be an object with one property defined as a method: subscribe. This method will have three arguments: options to supply data to include with the request to Mailchimp and success and error callbacks. The service is setup this way so that we can add more methods, related to the Mailchimp account or list, at a later time.

First, we check for an email address and throw an error if one isn't included. Next we create an object with the list ID and email address formatted in the structure the Mailchimp API expects.

We then make a call to the Mailchimp API’s subscribe method by passing the formatted request data as the first argument, and the success callback as the second argument. The third argument is the error callback, which we wrap in an anonymous function so that we can log it using Sails’s logger.


### Configuration

Our service is all setup, but we’ll want to update some configuration before we create the controller to handle the webhook requests. First up is to disable the blueprints API so no routes or controller actions are automatically generated. We want to be explicit and keep the API small.

The blueprints API configuration lives in “config/blueprints.js”:

```javascript
module.exports.blueprints = {
  actions: false,
  rest: false,
  shortcuts: false,
  pluralize: false,
  populate: false,
  autoWatch: false
}
```

We’ll also need to configure the route, which will handle the post request from Tito, in "config/routes.js”:

```javascript
module.exports.routes = {
  'post /tito': 'HooksController.tito',
};
```

The route handler will accept post requests to the “/tito” URL and it will call the “tito” action on the hooks controller.

Next, we’ll create “api/controllers/HooksController.js”:

```javascript
module.exports = {
  tito: function(req, res) {
    var webhookName = req.get('X-Webhook-Name');

    if ('ticket.created' === webhookName) {
      var email = req.body.registration.email;

      MailchimpService.subscribe({ email: email }, function (data) {
        sails.log.info('The email ' + email + ' is now subscribed to the newsletter');

        return res.ok();
      }, function (error) {
          return res.negotiate(error);
      });
    } else {
      return res.ok();
    }

  }
}
```

Controllers in Sails are objects with a function, defined as a property of the object, for each action. These functions have access to the request and response objects like route handlers in an Express.js app.

The “tito” action will gather data submitted by the Tito API in a format defined by the Tito documentation and then format it for use by the Mailchimp Service. Tito provides a variety of webhook requests, but we care about the request for ticket creation. To determine which Tito event has occurred we grab the “X-Webhook-Name” header value from the request.

If the webhook is for a “ticket.created” event, we can pull the email address out of the registration object of the request and call the subscribe method, we wrote earlier, on the Mailchimp service. The service we built allows for a success and error callback function as arguments, which we’ll use here to log a successful subscription to the console and return a successful 200 response with `res.ok()` or return the error message from the Mailchimp API with `res.negotiate(error)`. An optional step, included in this example, is to return a 200 response if the webhook is any type other than “ticket.created”.


Deploying the Application on Heroku
-----------------------------------

To get this server up and running on the free tier of Heroku, since it does not require a database or support for sessions, you will need to set the environment variables, and then trigger a build. Once you’ve created a new application on Heroku you can add the Mailchimp credentials:

```bash
heroku config:set MAILCHIMP_API_KEY=<api key> MAILCHIMP_LIST_ID=<list id>
```

After committing your work with get, all that’s left is to set the git url of your application and then push the code to trigger a build:

```bash
git remote add heroku <heroku url>
git push heroku master
```

From here your server should be running, which you can confirm by checking the logs for errors with `heroku logs`, and is ready for testing. To test it out you can create an event on Tito and add the URL of your webhook server with the “/tito” path and go through the ticket reservation flow.
