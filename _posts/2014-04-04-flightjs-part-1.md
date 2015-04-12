---
layout: post
title: Building a chat app with @flightjs Part 1 - Boarding Yeoman, Bower, Grunt (or Gulp), and Hogan
permalink: flightjs-part-1
category: Front end Ninja
tags: [flightjs, yeoman, bower, grunt, build tools]
---

This is the first post in a series on building a beautifully simple chat app with Flight.

The app will have 3 main views:

<img src="/public/img/flightjs1.png" alt="login view" style="width:100%;max-width:500px;" />
login view

<img src="/public/img/flightjs2.png" alt="conversation view" style="width:100%;max-width:500px;" />
conversations view (wait 3 seconds for the server to ping you)

<img src="/public/img/flightjs3.png" alt="chat view" style="width:100%;max-width:500px;" />
chat view

Check out my previous story to [get started with Flight](https://medium.com/@stefanritter/a-flight-js-hello-world-97fc4ae3072d). In these posts we will take Flight one extra passenger at a time. At the end of this tutorial I hope to have talked you through how to create this application with reusable Flight components on the front-end.

The full source code to the app we are going to write can be found here: [https://github.com/stefanRitter/flight-chat](https://github.com/stefanRitter/flight-chat)

And make sure to check out the app running away happily on Heroku:&nbsp;
[http://flight-chat.herokuapp.com/](http://flight-chat.herokuapp.com/)

I won’t talk you through each line of code (and I’ll mostly ignore the backend too), instead I’ll talk you through just enough exemplary pieces of the application, so you can easily piece together the rest.

Here’s an overview of what we’ll discuss:

Part 1:

*   setting up the workflow with **_Yeoman, Bower &amp; Grunt_**
*   using **_Hogan_** to precompile and render templates

[Part 2](/flightjs-part-2):

*   writing components with **_RequireJS_**
*   testing components with **_Karma &amp; Jasmine_**

[Part 3](/flightjs-part-3):

*   writing**_ data components_**
*   _**spying on jQuery.ajax**__&nbsp;**promises**_ to test data components


## Bower, Grunt/Gulp, &amp; Yeoman

First let’s set up our productivity &amp; workflow tools: Bower, Grunt and Yeoman. You’re going to need node &amp; npm installed for this, so make sure to visit [nodejs.org](http://nodejs.org/) and follow their installation instructions, if you haven’t got it yet. You will need super user (sudo) rights if you haven’t configured your permissions/directories ([checkout this article for more on that](http://stackoverflow.com/questions/18212175/npm-yo-keeps-asking-for-sudo-permission) and [this discussion if you’re on OSX](https://github.com/npm/npm/issues/3139)).

<pre>npm install -g grunt-cli bower yo</pre>

**_Bower_** is to front-end what npm is to Node and bundler is to Rails: It let’s you easily install, manage, and upgrade your front-end dependencies.

**_Grunt_** is a javascript task runner, similar to what rake is for Rails. Grunt will do whatever you want it to do, but usually you’ll use it to run your tests, minify and lint your javascript, and precompile any of your app’s assets. Similar to Grunt, [**_Gulp_**](http://gulpjs.com/) is a tool getting really popular recently, it utilizes Node’s streaming capabilities and has stricter plugin guidelines and rules than Grunt. The flight generator comes with Gulp out of the box, but I setup this tutorial in Grunt originally, so I decided to stick to Grunt for now. The repo has Grunt and Gulp setup, so you can choose which one to use. It won’t be complicated to make the switch once you're familiar with one of them. If you want to know more about Gulp, check out [this great podcast on Gulp, Grunt and Node streams.](http://javascriptjabber.com/097-jsj-gulp-js-with-eric-schoffstall/)&nbsp;I also wrote a [post on switching](http://blog.stefanritter.com/post/81871236665/switching-from-grunt-to-gulp).

Again if you’ve used Rails, you know it comes with a handy generator for anything from controllers to models, and database migrations. **_Yeoman_** brings this same convenience to Javascript based apps.

To create a new app, I usually start with a Yeoman generator and then customise it from there:

<pre>npm install -g generator-flight
mkdir flight-chat &amp;&amp; cd $_
yo flight flight-chat --skip-install</pre>

This will create our ‘flight-chat’ app in the flight-chat directory. Yeoman will ask you a few questions regarding Bootstrap and normalize.css, which in this case I negate, because I prefer to use my own styles using Sass with Compass. Notice the —skip-install flag on the last line, the flight generator uses Gulp as an alternative to Grunt so we’ll want to change the package.json a bit before running npm install.

**Setting up Grunt**

The first thing I want do is remove all Gulp dependencies from the newly created app, this includes the gulpfile.js file as well as the ‘gulp’ and ‘gulp-livereload’ entries in package.json. Then we can start adding Grunt, as well as a handful of Grunt tasks, which we’ll be using:

<pre>npm install grunt —save-dev
npm install grunt-contrib-jshint **grunt-templates-hogan** grunt-contrib-compass grunt-contrib-watch --save-dev
npm install load-grunt-tasks --save-dev</pre>

The final line installs a handy Grunt task loader, which we can use to load all our ‘grunt-’ modules at once, without having to specify them individually. Something you will appreciate once you’re in Gruntfile.js, which we will create next.&nbsp;

Gruntfile.js is a configuration file you setup in the root directory of your app, where it can sit nicely next to package.json, and bower.json — all the configuration files created by the Yeoman generator when creating the app. You can lookup the Gruntfile.js for Flight-chat in the Github repo to see how all the tasks are configured. As an example, I’ll talk you through setting up the Hogan task — the other tasks all work similarly, so you’ll easily get the hang of it from following this one example.

The Gruntfile.js is a Node module which gets required by Grunt when you execute a Grunt task from the command line. It exports one simple function into which Grunt injects itself through the grunt parameter:

<pre>module.exports = function(grunt) {
  // 1. load grunt modules
  // 2. configure tasks
  // 3. register tasks
};</pre>

A Gruntfile has three main sections, first you load all the Grunt modules, second you configure them, and finally third you register the main command line tasks you want Grunt to be able to execute.

Loading the Grunt related modules is a very simple one for us because we’re using the previously installed load-grunt-tasks module;

<pre>// 1. load grunt modules
require('load-grunt-tasks')(grunt);</pre>

I’ll talk about the configuration step in the next section. Let’s first have a look a the two tasks we’ll be using — defined in the end of the Gruntfile.js

<pre>// 3. register tasks
grunt.registerTask('default', ['watch']);
grunt.registerTask('build', ['jshint', 'compass', 'hogan']);</pre>

The default task will run whenever you evoke simply ‘grunt’ from the command line, we have set it up to call the 'watch' task which monitors our files for changes, calling the appropriate tasks for a registered change.
The build behaviour we give Grunt will ask it to check our Javascript files with Jshint, compile our sass files with compass and finally generate our templates using Hogan.

## **Templating** with Hogan

A Hogan template looks just like HTML, except that all the dynamic content is marked in curly braces &#123;&#123; — like so:

<pre>&lt;div class="chat-message-text"&gt;&#123;&#123;text&#125;&#125;&lt;/div&gt;</pre>

Our flight-chat app uses five rather simple templates - but if you want to know more about Hogan, including the more advanced features like loops and conditionals checkout the [mustache docs](http://mustache.github.io/mustache.5.html), which define the syntax Hogan follows.

To add Hogan to our project simply install it with bower…

<pre>bower install hogan —save</pre>

&nbsp;… and then add it to to bottom of our app/index.html file:

<pre>&lt;!--[if lt IE 9]&gt;
 &lt;script src="/bower_components/es5-shim/es5-shim.js"&gt;&lt;/script&gt;
 &lt;script src="/bower_components/es5-shim/es5-sham.js"&gt;&lt;/script&gt;
&lt;![endif]--&gt;
&lt;script src="/socket.io/socket.io.js"&gt;&lt;/script&gt;
&lt;script src="/bower_components/jquery/dist/jquery.js"&gt;&lt;/script&gt;
**&lt;script src="/bower_components/hogan/web/builds/2.0.0/hogan-2.0.0.min.js"&gt;&lt;/script&gt;**
&lt;script src="/bower_components/requirejs/require.js" data-main="js/main.js"&gt;&lt;/script&gt;</pre>

There are two steps involved when using templates: Templates first have to get compiled, this is when Hogan converts them from HTML text into a Javascript function.
And then in the second step we use this compiled version of the template in order to render it to the page with Flight.

We’ve already installed the Grunt task which we’ll use to compile our templates as part of our build process, so let’s configure it in Gruntfile.js:

<pre>// 2. configure tasks
grunt.initConfig({
 yeoman: {
  app: 'app',
  dist: 'dist'
 },
 pkg: grunt.file.readJSON('package.json'),</pre>
<pre> hogan: {
  publish: {
   options: {
     prettify: true,
     amdWrapper: true
   },
   files:{
    '&lt;%= yeoman.app %&gt;/js/templates.js': ['templates/**/*.html']
    }
   }
  }
});</pre>

A few things are being setup here: Under the yeoman property,&nbsp;we first define the main folders of our app (which in our case is ‘app’) and a general destination folder (we’re not using dist, because our server simply serves everything directly from the app folder).
The ‘pkg’ property creates an object out of our npm package.json file, so we could reference it later.
And finally we define our hogan task. [Check out the docs](https://github.com/vanetix/grunt-templates-hogan) to see what options this task comes with. Important is that we wrap our generated template functions into an AMD module — so it integrates nicely with Flight through RequireJS later on. Under the files definition you can see that we are taking all html files from the templates/ folder and merging them into one templates.js file.
‘templates.js’ contains an object, who's keys are named after the file path of the original template — here’s an example of how we are going to use it in our app to render views from the templates in templates.js:

<pre>var templates = require('js/templates'),
    htmlString = templates['templates/app_view.html'].render();</pre>
<pre>$('#app').html(htmlString);</pre>

If our template would have dynamic content — e.g. &#123;&#123; variableName &#125;&#125; — we could pass in the dynamic values to the render function like so:&nbsp;

<pre>var htmlString = templates['templates/app_view.html'].render({
  variableName: ‘dynamic content inserted at render time’
});</pre>

In the previous example above you saw already the use of RequireJS — we called **require()** to get the original templates.js file which Hogan generated for us. RequireJS the final big Flight passenger we need to understand before we can focus our attention on Flight itself.

[**goto Part 2 - boarding RequireJS, Karma, and Jasmine**](/flightjs-part-2)
