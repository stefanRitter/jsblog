---
layout: post
title: Switching from Grunt to Gulp
permalink: switching-from-grunt-to-gulp
category: Front end Ninja
tags: [gulp, grunt, build tools]
---

After publishing my [tutorial on writing a chat app with flight](http://blog.stefanritter.com/post/81767869139/building-a-chat-app-with-flight-part-1-boarding), I got a few requests on switching from Grunt to Gulp.

I decided to put them both in the repo to compare. In total the gulpfile is 23 lines shorter than the Gruntfile.

Because it’s a tutorial application the Grunt tasks are rather straightforward, no minifying, concatting or optimisation going on. Which means we won’t really take advantage of Gulp’s streaming features. Streaming allows you to move data from task to task, so we could lint, concat, and uglify in one continuous stream - without having to save interim results somewhere.

Our Gruntfile.js has a Jshint task to lint Javascript, a Karma task to run tests, Hogan is used to pre-compile templates, and Compass generates one ‘application.css’ file out of all my sass files. Add to that one watch task and we’re all setup.

[Check out the Gruntfile.js here](https://github.com/stefanRitter/flight-chat/blob/master/Gruntfile.js). That’s 85 lines of code. Compared to 62 for the [gulpfile.js](https://github.com/stefanRitter/flight-chat/blob/master/gulpfile.js). What’s nice about Gulp is that the setup follows a more node-style syntax. Here’s a look at the structure of Gruntfile:

{% highlight js %}
module.exports = function(grunt) {
  'use strict';

  // load all grunt tasks
  require('load-grunt-tasks')(grunt);

  // configuration
  grunt.initConfig({
    yeoman: {
      app: 'app',
      dist: 'dist'
    },
    pkg: grunt.file.readJSON('package.json'),

    jshint: {
      options: {
        jshintrc: '.jshintrc'
      },
      all: [
        '*.js',
        'server/**/*.js',
        'test/**/*.js',
        '<%= yeoman.app %>/js/**/*.js',
        '!<%= yeoman.app %>/js/templates.js',
        '!<%= yeoman.app %>/js/mixin/with_quick_hash.js'
      ]
    }
    //...
  });

  // tasks
  grunt.registerTask('default', ['karma:unit:start watch','watch']);
  grunt.registerTask('build', ['jshint', 'compass', 'hogan']);
};
{% endhighlight %}

And when you compare that to the structure of a gulpfile…

{% highlight js %}
'use strict';

var gulp = require('gulp'),
    compass = require('gulp-compass'),
    hogan = require('gulp-hogan-compile'),
    jshint = require('gulp-jshint'),
    karma = require('gulp-karma');

var paths = {
  scripts: [
    '*.js',
    'server/**/*.js',
    'test/**/*.js',
    'app/js/**/*.js',
    '!app/js/templates.js',
    '!app/js/mixin/with_quick_hash.js'
  ],
  sass: 'sass/**/*.sass',
  templates: 'templates/**/*.html',
  tests: 'test/**/*.js'
};

gulp.task('lint', function() {
  gulp.src(paths.scripts)
    .pipe(jshint())
    .pipe(jshint.reporter('default'));
});
{% endhighlight %}

… you’ll see the typical node module syntax. Grunt’s syntax in contrast has this slightly awkward, giant object literal in the middle of it.

To be fair though, there are some gotchas with Gulp, but they are mostly plugin specific. The hogan plugin behaves differently than the Grunt equivalent in two ways. Firstly it names templates differently, hence the extra lines of code for the templateName function to mitigate the naming differences. And secondly, it relies on Hogan 3 which is not really published to npm or the browser version of hogan.

The second problem is karma integration, in Grunt it’s easy to just have Karma started in a separate thread and then trigger the tests when changes are detected. gulp-karma just isn’t there yet, but it’s progressing quickly - [check it out on github](https://github.com/lazd/gulp-karma/).
