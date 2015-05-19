---
layout: post
title: Non-RESTful Actions Mixin for Ember Data Model
---

## Background

I was recently working on some functionality on one of our Ember applications at work that involved making non-RESTful calls to our API. One of our models can be dismissed by making a POST request to a nested route `/dismiss`. We’re using Ember Data, which makes the assumption that all of your requests will be RESTful, so the dismiss functionality will require a bit of work.

The quickest way to implement the dismiss call, would be to use `jQuery.ajax` and construct the URL using the model’s ID. However, this presents a few challenges: the Ember work flow when requesting data uses promises (but `jQuery.ajax` uses callbacks), the constructed URL is brittle and is not flexible to URL changes for the model or API, and the code written with `jQuery.ajax` is not very reusable on other controllers or routes.

My next step was to do some research to see if anyone else had documented how they dealt with non-RESTful API calls. I figured it was a common problem and I came across this great post by [Chris Ball](http://cball.me) on making [Non-RESTful API calls with Ember Data](http://cball.me/non-restful-api-calls-with-ember-data/). In the post Chris details why you shouldn’t use `jQuery.ajax`, or more specifically `Ember.$.ajax`, to complete the action and how you can take advantage of Ember adapters instead.


## Creating a Reusable Solution

My goal was to craft a solution where the code not specific to this model could be reused. The solution I went with was to create an Ember Mixin that could be extended by any model that had non-restful actions.

The mixin ended up with two functions as properties: `getActionUrl` and `nonRestAction`.

{% highlight javascript %}
import Ember from 'ember';

export default Ember.Mixin.create({
  nonRestAction: function (action, method, data) {
    const type = this.constructor.typeKey;
    const adapter = this.container.lookup(`adapter:${type}`);

    return adapter.ajax(this.getActionUrl(action, adapter), method, { data: data });
  },

  getActionUrl: function(action, adapter) {
    const type = this.constructor.typeKey;
    const id = this.get('id');

    return `${adapter.buildURL(type, id)}/${action}`;
  }
});
{% endhighlight %}

The first method, `getActionUrl`, uses the model’s adapter to construct the URL for the request. This is valuable because it means the model can change or have custom adapter behaviour and the correct URL will still get constructed. This URL is created by getting the model name from the constructor function’s `typeKey` property and passing this into the adapter’s `buildUrl` method along with the model’s ID, and then appending the action to this root URL.

The second method, `nonRestAction`, gets the current model’s adapter by looking up the container using the type key, uses the `getActionUrl` method to construct the URL and then returns a promise for the request. This is accomplished by using the adapter’s ajax method and passing along the method and data, which are allowed as arguments.


## Using a Model with the Mixin

To setup a model to use the mixin: import the mixin in the desired model file and provide the mixin as the first argument when defining the model by extending an Ember Data model or Ember Object. The example below details how you could update the model when a non-restful action is successful.

{% highlight javascript %}
import DS from 'ember-data';
import NonRestAction from 'coaching-assistant/mixins/non-rest-action';

export default DS.Model.extend(NonRestAction, {
  dismiss: function() {
    const type = this.constructor.typeKey;

    return this.nonRestAction('dismiss', 'POST').then((result) => {
      this.store.pushPayload(type, result);
    });
  }
});
{% endhighlight %}


## Calling a Non-RESTful Action

Calling one of these actions from somewhere else in your application can be accomplished by calling a defined method on the model (setup in the previous example code as the dismiss property) or by calling the `nonRestAction` method with all the action and method arguments, and optional data (if required).

{% highlight javascript %}
import Ember from 'ember';

export default Ember.Route.extend({
  actions: {
    dismiss: function () {
      this.get('model').dismiss().then((response) => {
        // Success

      }, function (error) {
        // Failure
        console.error(error);
      });
    }

    focus: function () {
      this.get('model').nonRestAction('focus', 'GET');
    }
  }
});
{% endhighlight %}

The first action, dismiss, shows an example of using a non-RESTful action that has been defined on the model, with custom behavior setup to update the model data when the request completes. It then has code in this route’s action to handle success and failure.

The second action, focus, simply makes a GET request for an action called focus and doesn’t bother handling the request promise in any way.


## Conclusion

This mixin allows for a consistent pattern when making Ajax requests that are associated with your models, but don’t fit in with Ember Data’s strict expectation of RESTful API endpoints. It allows you the flexibility of renaming a model or changing an adapter and having the correct URL and corresponding promise request created.
