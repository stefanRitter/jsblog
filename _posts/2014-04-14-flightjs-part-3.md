---
layout: post
title: Building a chat app with @flightjs Part 3 - boarding Jasmine Spies
permalink: flightjs-part-3
category: Front-end Ninja
tags: [flightjs, jasmine, testing, promises]
---

This is part three in my series on building [this chat app](http://flight-chat.herokuapp.com/) with Flight, check out [part 1 here](/flightjs-part-1) and [part 2 here](/flightjs-part-2). The source code is [available on github](https://github.com/stefanRitter/flight-chat).

In this final post I want to discuss writing and testing data components, which persist JSON data to the server. To do this we will have to test our component by spying on jQuery.ajax in our Jasmine unit test.

The component we’re going to write is called ‘authenticate_user’ located in the component_data/ folder. We want to use this component to handle the ajax calls to the server when the user is attempting to log in. If the user is logged in successfully we want to switch to main app page, otherwise we will trigger a failed event and add the reason for the failure into the event so that the ui can print it to the page. Let’s define this behaviour in a spec:

<pre>describeComponent('component_data/authenticate_user', function () { </pre>
<pre>  it ('should trigger uiFormError on failed login', function ()   {
     ....
     expect('uiFormError').toHaveBeenTriggeredOn(...);
  });</pre>
<pre>  it ('should trigger uiSwitchPage on successful login', function () {
    ....
    expect('uiSwitchPage').toHaveBeenTriggeredOn(...);
 });</pre>
<pre>});</pre>

That seems pretty straight forward. This is what our component looks like:

<pre>
define(function (require) {
 'use strict';</pre>
<pre> var defineComponent = require('flight/lib/component');</pre>
<pre> return defineComponent(authenticate);</pre>
<pre> function authenticate() {
   // attributes
   this.defaultAttrs({
     submitButtons: 'input[type=submit], button[type=submit]'
   });</pre>
<pre>   this.authenticateUser = function(e, data) {
     var _this = this,
     formData = data.formData,
     name = formData[0].value;

     if (!name) {
       return this.trigger(this.select('submitButtons'), 'uiFormError', {error: 'invalid name'});
     }</pre>
<pre>     $.ajax('/app/login', {
       method: 'POST',
       data: formData
     })
     .done(function(data) {
       if (data.error) {
         _this.trigger(_this.select('submitButtons'), 'uiFormError', {error: 'unknown error please try again'});
       } else {
        window.__APP.__USER = data.user;
        _this.trigger('uiSwitchPage', {name: 'appPage'});
      }
   })
    .fail(function(err){
      _this.trigger(_this.select('submitButtons'), 'uiFormError', {error: 'unknown error'});
    });
  };</pre>
<pre>   // initialize
   this.after('initialize', function () {
     this.on('dataUserLogin', this.authenticateUser);
   });
 }
});</pre>

We can see our component listens for the ‘dataUserLogin’ event. And it will return uiFormError if the user hasn’t provided a name or the server has responded unexpectedly — otherwise it will trigger uiSwitchPage to make the app enter it’s main view. Note: there’s one line where the user object is set to a property of the __APP global, this is probably not the best way to share data between components and I might refactor this in the future (putting the user data into it’s own data component).

Notice how we are using the jQuery promise syntax for our AJAX call? Instead of providing .ajax() with success and failure callbacks we are assigning them to the return value of .ajax() — check out the jQuery [deferred interface docs](https://api.jquery.com/category/deferred-object/) for more information.

The beauty of the promise syntax is that it makes for very clean testing code. If we want to complete our unit test for the authenticate component we will have to prevent our test from attempting to call the server. This would most likely fail and break our test because our server is either not running or the test client is not authenticated. If we have a continuous integration system setup, which runs our tests somewhere else, it would also not have access to our development server. The second reason is that it would make our tests run significantly slower, because we’d have to wait for an AJAX timeout and accommodate asynchronous testing.

Also we don’t really want our tests to be testing jQuery. We can safely assume jQuery not to be the problem in our code (they have their own test suite anyways). But more importantly we want to make sure that our tests are isolated specifically around the code we wrote ourselves. Go and check out [Misko Hevery’s blog](http://misko.hevery.com/) — he has brilliant resources on testable code, including a great guide on how to write better testable code.

For us to solve the ajax call issue, Jasmine comes with a spyOn method which we can use to replace the external dependencies of our code. This will allow us not only to prevent our test from trying to call the server, but it will also give us complete control over what our component will get in return from its call to jQuery.ajax() during the tests. Remember we want to test both the successful as well as the failed login behaviour. Here’s how we would setup spying on jQuery.ajax() in our test suite:

<pre>var d;</pre>
<pre>beforeEach(function() {
   spyOn(jQuery, 'ajax')**.andCallFake**( function() {
   d = $.Deferred();
   return d.promise();
 });
});</pre>

Before each of our tests we want Jasmine to replace the original jQuery.ajax() code with the one we have provided above. So when our component now tries to call the server it will immediately get our own promise d returned — which was created in our fake replacement function.

With access to our own promise object, we can now fully control the next step our component has to take: If we want to test how our component responds to a failed ajax call we can use [d.reject()](https://api.jquery.com/deferred.reject/) and if we want to simulate a successful login we can do so with [d.resolve()](https://api.jquery.com/deferred.resolve/). Again check the jQuery docs for more on these functions.

Let’s see what our test looks like now for a failed login:

<pre>var d;</pre>
<pre>beforeEach(function() {
 spyOn(jQuery, 'ajax').andCallFake(function() {
   d = $.Deferred();
   return d.promise();
 });
 **setupComponent();**
});</pre>
<pre>it ('should trigger uiFormError on failed login', function () {
   spyOnEvent(document, 'uiFormError');
   this.component.trigger('dataUserLogin', {
     formData: [{value: 'username'}]
   });
   **d.reject();**
   expect('uiFormError').toHaveBeenTriggeredOn(document);
});</pre>

In the beforeEach function we do two things, spy on jQuery.ajax() to replace it with our own function, and create the component with a call to setupComponent().&nbsp;

Next in our test we want to see if uiFormError is triggered correctly when a login attempt fails. First we ask our component to try to login a user by triggering dataUserLogin with the username ‘username’. And right after this is where the neat little call to d.reject() is used. This should let our component execute it’s fail() logic.
If we wanted to get the user to login successfully on the other hand we would write:

<pre>d.resolve({user: {data: 'userData'}});</pre>

This gives our component the go ahead to execute the done() logic passing in our fake user object (which we assume a server would return) for it to process.

To see the full test code checkout the [full spec](https://github.com/stefanRitter/flight-chat/blob/master/test/spec/component_data/authenticate_user.spec.js) and the [component definition](https://github.com/stefanRitter/flight-chat/blob/master/app/js/component_data/authenticate_user.js) on github. Thanks for reading!
