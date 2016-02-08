---
layout: post
title: Starting a Polymer Project
permalink: starting-a-polymer-project
category: Front-end Ninja
tags: [polymer, yeoman, bower]
---

**Finding your way in the yo generator, seed-element, and boilerplate jungle**

<img src="/blog/assets/img/polymer.gif" alt="Polymer" style="width:100%;max-width:464px;" />

When I was about to start with my first Polymer app, I was faced with the typical questions: How do I structure the code? Where would my components go? Where is the app entry point, and what Grunt tasks best to add to it?

Before we start, make sure you have a basic understanding of yeoman, grunt, and bower, best places to start: [http://yeoman.io/](http://yeoman.io/),&nbsp;[http://bower.io/](http://bower.io/)&nbsp;and&nbsp;[http://gruntjs.com/](http://gruntjs.com/)

***** Update:** The Yeoman generator was just updated to create a 'classical' app structure instead of defaulting to the relatively positioned seed element. Make sure you install the newest version of the generator by pulling it directly from github, like so:

<pre>npm install -g git+https://github.com/yeoman/generator-polymer.git</pre>

These are the possible app starting points I came a cross on the interwebs:

1.  [yo polymer](https://github.com/yeoman/generator-polymer)&nbsp;(same as polymer:app)
2.  [yo polymer:el&nbsp;your-element](https://github.com/yeoman/generator-polymer)
3.  [yo polymer:seed](https://github.com/yeoman/generator-polymer)
4.  [yo polymer:gh](https://github.com/yeoman/generator-polymer)
5.  [yo element](https://github.com/webcomponents/generator-element)
6.  [yo element:repo](https://github.com/webcomponents/generator-element)
7.  [https://github.com/PolymerLabs/seed-element](https://github.com/PolymerLabs/seed-element)
8.  [https://github.com/webcomponents/polymer-boilerplate](https://github.com/webcomponents/polymer-boilerplate)
9.  Example app repos: [Topeka App](https://github.com/Polymer/topeka) &amp; [Paper Calculator App](https://github.com/Polymer/paper-calculator)

Of course, in the end I ended up choosing a bit of a mix of pretty much all of them. Let’s have a look at how they differ and can be combined.

## Using the Yeoman Polymer Generator &amp; and the seed-element

The yo polymer:seed generator is optimised around producing a single component in full isolation with its own repo. You can’t really use it to scaffold out an entire app — at least as you would come to expect from other generators like the one for AngularJS (but afterall what is an app if not just another web component?). If you want to scaffold a more classical app structure yo polymer:app and yo:polymer:el are for you (scroll down for more on those).

There are two ways you will run the generator to create a polymer seed element — don’t run the generator just yet, as it uses relative paths for the dependencies, bare with me just a moment on that:

<pre>yo polymer:seed
yo polymer:gh</pre>

yo polymer:seed produces a fresh new component with the core-component-page setup ready for you to demo your component to the world. This component is essentially the same as the seed-element which you can download as a zip file from [Polymer’s main site.](http://www.polymer-project.org/docs/start/reusableelements.html)

yo polymer:gh on the other hand can be used to publish your existing component to [github pages](https://pages.github.com/) by cloning it onto the gh-pages branch and pushing it up to github for you. The command will package all your dependencies into a single components/ folder within the gh-pages branch. This will fail if you haven’t got a repo by the same name as your component. More on this later when we have a look at the [polymer-boilerplate](https://github.com/webcomponents/polymer-boilerplate).

**Note:** Before you run the generator, be sure to have the right folder structure set up. It will run bower install assuming your new element is one of many in a parent project folder. As a result bower will place the dependencies into the preceding folder. For example if you’re running the generator in ~/myproject bower will install polymer, platform and core-component-page into your ~home directory.

**Getting the directory structure setup**

The generator creates a new component and not an app per se. This seems to fit with Polymer’s philosophy: instead of writing an app you are encouraged to write isolated, independent components which sit in their own repo and can be published independently from your app. As a result the following directory structure works best for the generator:

<pre>all-my-components/
  my-first-component/</pre>

If you create your top level directory and then cd into it, you create your second folder named after your component there. cd into this folder, and finally that’s where you can run the generator:

<pre>**~/all-my-web-components/my-first-component
$ yo polymer**</pre>

After the generator has run (it will ask you for your github name and component name) your directories will look like this:

<pre>all-my-components/
  platform/
  polymer/
  core-component-page/
  my-first-component/
    .bowerrc
    my-first-component.html
    ...</pre>

Notice how platform and polymer are in the parent directory to my-first-component?
If you look inside my-first-component.html you can see the dependencies are imported via the relative link ../platform/platform.js as defined in .bowerrc

## Using the Polymer boilerplate &amp; yo element

The yo polymer generator is great for creating standalone components, but I don’t necessarily want to litter my github with every little component I write (although I have a feeling I will end up ripping apps apart and place components into their own repo to reuse them between apps and show them to the world in the near future). On the contrary the [polymer-boilerplate](https://github.com/webcomponents/polymer-boilerplate) repo sets you up with a proper 'old-school' app structure.

The boilerplate is based on a similar directory structure to the one suggested on the polymer website under [‘creating elements &gt; the basics’](http://www.polymer-project.org/docs/start/creatingelements.html):

<pre>your_app/
 bower_components/  &lt;&lt; all dependencies
 components/        &lt;&lt; all your project specific components
 index.html
 bower.json
 ...</pre>

This allows for the more traditional app feel, all the dependencies are nicely tucked away in bower_components, which can be safely added to .gitignore. Under your_app you can also define a Gruntfile and add your favourite build processes (and the polymer web component specific [vulcanize](https://github.com/Polymer/vulcanize)).
Also keep in mind that any components can be added via bower - including ones you have previously published yourself (using the yo polymer generator). You can simply pull them like so:

<pre>bower install --save yourGithubName/yourComponentsRepo</pre>

If you have a look in the [boilerplate’s repo](https://github.com/webcomponents/polymer-boilerplate) you will see a slightly different folder structure, but in essence it is the same as described above. Instead of components/ you custom elements are put into src/.

Additionally to the bower- and your custom components the boilerplate repo also comes with a dist/ folder. If you have a look in the boilerplate’s Gruntfile.js you can see a bunch of taks defined there which use this folder. In short all bower components and your custom compontens are copied over into dist/, then the replace task makes sure all relative paths are updated to the components’ new location. All this is necessary to do the third task, called ‘deploy’ which will produce the same output you would get from running yo polymer:gh. Namely, it copies the files to a temporary location, creates gh-pages branch and pushes them up to origin, so that github can setup a project page for you.

Alternatively instead of forking/cloning/downloading the boilerplate repo you can also simply use the [yo element generator](https://github.com/webcomponents/generator-element). It’s similar to plolymer’s generator but is framework agnostic, so it will ask you what kind of web component you want to create (polymer, x-tab, or plain old vanilla JS). To create an app-like structure similar to the boilerplate you can run:

<pre>yo element:repo</pre>

Once you’re ready to create your first element within the src/ folder simply cd into it and run:

<pre>yo element</pre>

Which will scaffold out a plain html page with your component’s name. Not as handy as the yo polymer generator I must admit, but at least you have full control over your app’s components that way.

Since it's last update you can now also use the polymer generator for this kind of workflow. yo polymer will scaffold your app, and with yo polymer:el you can build out new elements quickly within it.

## Conclusion

Use the Polymer generator to create repos for your standalone components based on the seed-element. It also allows you to publish them easily individually with a default component demo page. But make sure you're in a subfolder of a parent directory which can host your dependencies.

Use the polymer-boilerplate and the yo element generator to create one main repo for your app. It will optionally setup some Grunt tasks. You can take it anywhere from there and manage other components conveniently with bower and the bower_components folder.&nbsp;Since it's last update you can now also use the polymer generator for this kind of workflow. yo polymer will scaffold your app, and with yo polymer:el you can build out new elements quickly within it.

I’ll leave it up to you to dig into the Topeka demo app’s structure, but it’s essentially one repo for the published components and a separate repo for the app’s own build process, which pulls in the components via bower.

Topeka also uses the [Vulcanizer](https://github.com/Polymer/vulcanize) to concat all your web components together, which reduces the number of HTTP requests needed to construct your web app. More on the Polymer/vulcanize can be found on the [Polymer website in this article by Addy Osmani](http://www.polymer-project.org/articles/concatenating-web-components.html).
