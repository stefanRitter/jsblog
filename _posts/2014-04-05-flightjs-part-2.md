---
layout: post
title: Building a chat app with @flightjs Part 2 - boarding RequireJS, Karma, and Jasmine
permalink: flightjs-part-2
category: Front-end Ninja
tags: [flightjs, karma, jasmine, testing, requirejs]
---

This is part three in my series on building&nbsp;[this chat app](http://flight-chat.herokuapp.com/)&nbsp;with Flight, check out&nbsp;[part 1 here](/flightjs-part-1)&nbsp;and&nbsp;[part 3 here](/flightjs-part-3). The source code is&nbsp;[available on github](https://github.com/stefanRitter/flight-chat).

## **Modules with RequireJS**

[RequireJS](http://requirejs.org/) is a module system for Javascript, wich makes it easy to write encapsulated and reusable code. RequireJs modules can then be dynamically required only when needed.
There are other methods of modularising Javascript, [this is a blogpost by Addi Osmani](http://addyosmani.com/writing-modular-js/) comparing AMD style modules like RequireJS to CommonJS-style modules.

Understanding RequireJS is very straightforward — all you need to know is the two functions **define()** and **require()**.&nbsp;
Yeoman has already installed and added RequireJS to our app/index.html so we won't need to add it to our app manually. Also take a look in app/js/ and you will see the main.js file which has a small piece of code to configure RequireJS (mainly setting up some shortcuts to the paths in our app, and calling the first require() to get Flight’s own library).

First let’s define a barebones module in a file called simpleModule.js and we can save it under app/js/mixin:

<pre>define(function() {
  console.log('in my simple module');
});</pre>

We can then make a call to require in main.js, and see output in the console:

<pre>require('mixin/simpleModule');</pre>

There are two optional parameters that can be passed into a call to define(). They can change the module definition’s signature quite a bit. One parameter is for the module's name, and the other is an array to list the module’s dependencies:

<pre>// 1 — giving a module a name
define('myNamedModule', function() {
  return {name: 'myModule'};
});</pre>
<pre>// 2 — defining a module's dependencies and giving it a name
define('moduleDependingOnJquery', ['jQuery'], function($) {
  return $('button');
});</pre>
<pre>// 3 — defining a module's dependencies without a name
define(['underscore', 'jQuery'], function(_, $) {
  var array = [2,3,4,2,3,4,6];
  return _.uniq(array);
});</pre>
<pre>// 4 — asking RequireJS to parse parameters for dependencies
define(function(underscore, jQuery) {
  var _ = underscore,
       $ = jQuery;
});</pre>

RequireJS looks for each dependency defined in the array and injects them into the module’s function as its parameters. In example 3 our module has access to underscore’s uniq function through the ‘_’ parameter and it could use jQuery through the usual dollar-sign syntax.&nbsp;
Compare that with example 4 where we ask RequireJS to load dependencies based on parameter names — you can see we needed an extra var expression to get the same short access to these two libraries as a result.

If you don’t want to list a module’s dependencies right in the beginning when calling define(), you can do so later within the module by using **require()**:

<pre>// 5 — using require()
define(function(require) {

 var $ = require('jQuery');

 var templates = require('js/templates'),
      template = templates['templates/app_view.html'].render();

 return function() { $('#app').html(template); };
});</pre>

Notice that we had to ask for require as a dependency first before we could start using it.

Now you know how to define and require modules, and how to dynamically load pre-compiled Hogan templates and render them! Finally, let’s write some Flight components.

## Good Karma with Jasmine tests

Our chat app has three Flight.js pages located in app/js/page: The first one is the default page which get’s called when we enter the app for the first time. It attaches three basic components to the document, one of which checks if the user is already logged in and triggers a ‘uiSwitchPage’ event. In that event it tells the app to either go to the login page or the app page next.&nbsp;
Let’s write the switchPage component, which listens for this event.

We start with calling our yeoman generator in the command line:

<pre>yo flight:component switchPage</pre>

This generates two files for us, one being for the component itself and one for its tests. The generator places all components into the app/js/component/ folder, but I prefer to split them into ui vs data components, so I’ll move this one into app/js/component_ui/ as switchPage handles user interface events.

Our spec file will look like the following after generation — notice Flight uses [Jasmine](http://jasmine.github.io/1.3/introduction.html) for its tests, which is a behaviour driven testing framework, so it should be very straightforward - like reading in plain English - what the tests are about:

<pre>'use strict';
describeComponent('**component_ui**/switch_page', function () {</pre>
<pre> // Initialise the component and attach it to the DOM
 beforeEach(function () {
   setupComponent();
 });</pre>
<pre> it('should be defined', function () {
   expect(this.component).toBeDefined();
 });</pre>
<pre> it('should do something');</pre>
<pre>});</pre>

You can run the test already in the command line with ‘npm test’, which will fire up karma and run our tests with PhantomJS (a command line webkit browser).&nbsp;Because the Yeoman generator created an empty component already our test should pass just fine.

Our switchPage component only listens to one event, which we called ‘uiSwitchPage’. We can define it’s behaviour as follows — replacing all functions within the describeComponent() of the auto-generated spec:

<pre>it('should initialize a new page on uiSwitchPage', function() {
  setupComponent();

  // 1. spy on the page's initialize function
  // 2. trigger event on component
  // 3. assert the page was initialized
});</pre>

Before we trigger the event on our component we need to ask Jasmine to spy on the signInPages initialize function, so that we can be sure the component called it. After that we can trigger the switchPage event and let Jasmine check if our component has reacted accordingly by initializing the page. The final test will look like this:

<pre>describeComponent('component_ui/switch_page', function () {
 it('should initialize a new page on uiSwitchPage', function() {
   setupComponent();

   **spyOn(this.component.attr, 'signinPage');**
   this.component.trigger('uiSwitchPage', {name: 'signinPage'});
   expect(this.component.attr.signinPage).toHaveBeenCalled();
 });
});</pre>

On the bold line you can see that we are planning to save the page’s initialize function as a default attribute in our component. We are then using Jasmine’s spyOn() method to be able to test if it was successfully called. When we run this again with npm test, obviously our test will fail.

Here is the implementation of the switchPage component, based on the above test:

<pre>define(function (require) {
 'use strict';

  var defineComponent = require('flight/lib/component'),
       **&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;appPage = require('page/app_page'),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;signinPage = require('page/signin_page');**

 return defineComponent(switchPage);

 function switchPage() {
   // attributes
   this.defaultAttrs({
**&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;'signinPage': signinPage,
       'appPage': appPage**
   });

   // initialize
   this.after('initialize', function () {
     this.on('uiSwitchPage', function(e, page) {
**&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;this.attr[page.name]();**
     });
   });
 }
});</pre>

In bold you can see the three main steps this component executes. First it requires our app’s two pages, after which it saves each of them to the component's default attributes, and finally it calls the pages based on the page.name value coming from the ‘uiSwitchPage’ event.

Run the test again, and it should all pass nicely.

[Goto part 3 here.](/flightjs-part-3)
