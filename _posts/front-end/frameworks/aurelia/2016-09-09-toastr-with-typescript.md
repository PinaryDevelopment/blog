---
layout: post
permalink: /frontend/frameworks/aurelia/toastr-with-typescript
title: Aurelia and toastr using Typescript
excerpt: How to utilize toastr in Aurelia with Typescript typings.
categories: ['Aurelia', 'toastr', 'Typescript', 'aurelia-cli']
---

As mentioned in the previous post, getting to know Aurelia has been fun, but not without its challenges. I would like to journal some of those challenges and the solutions I've come up with to date. The current third-party library that I would like to discuss incorporating into Aurelia is [toastr](http://codeseven.github.io/toastr/).

Toastr is a very neat little library that handles displaying colorful non-blocking notifications on your webpage. Integrating this library into an Aurelia application proved a bit more challenging than I had anticipated and since I've noticed that others have had similar difficulties, I wanted to document it here for the benefit of others.

----
*__Note__: Both this post and the previous post present solutions for incorporating libraries into the Aurelia framework that have dependencies on the jQuery library. Ideally, the powerful binding/templating engine that Aurelia provides should be utilized to their fullest and jQuery shouldn't be required as it provides a large bloat(84kB) to the final deliverable. Unfortunately, familiar and more widely used tools often make for a quicker development process and in many cases can remove a number of potential bug fixes or desired improvement implementation as the library has been more thoroughly vetted by the community at large. As the Aurelia community continues to grow and the framework continues to evolve and harden, I expect to see more libraries that will harness more of the framework's power and provide some of these pieces of functionality without the need to rely on jQuery.*

----

After creating a new project utilizing the cli, there are two npm packages that need to be installed.

`npm install jquery toastr --save`

Then the associated typings files need to be installed as well.

`typings install dt~jquery dt~toastr --global --save`

These installs will allow you to utilize these libraries in Typescript, but at the moment, things will fail transpilation if you run `au build`. The reason for this is that there is a variable name declared in the angular-protractor index.d.ts file that conflicts with a declaration of the same name in jquery's .d.ts file. In order to fix this, I needed to comment out line 1839(`declare var $: cssSelectorHelper`) in `typings/globals/angular-protractor/index.d.ts`.

Following this, you need to add two entries in the `aurelia_project/aurelia.json` file. Anywhere in the `bundles.dependencies` section as follows

{% highlight json linenos=table %}
"jquery",
{
  "name": "toastr",
  "path": "../node_modules/toastr",
  "main": "toastr",
  "resources": [
    "build/toastr.min.css"
  ],
  "deps": ["jquery"]
}
{% endhighlight %}

*__Note__: With third-party libraries, I try to configure the `aurelia.json` file with the minified versions of the third-party libraries. In this case, there is something that causes toastr's `.min.js` file to not work properly with the module loader, so to get the library to work in Aurelia, I needed to point it at the un-minified version of the javascript file.*

Now you should be able to use the library in any of your view/view models as follows(I'm demonstrating with my `app` files, but this could be done from any of your views).

*`app.html`*
{% highlight html linenos=table %}
<template>
  <require from="toastr/build/toastr.min.css"></require>

  <button click.delegate="showToast()">Click</button>
</template>
{% endhighlight %}
----

*`foo.ts`*

{% highlight typescript linenos=table %}
import toastr = require('toastr');

export class Foo {
  attached() {
    toastr.warning('here');
  }

  showToast() {
    toastr.info('here');
  }
}
{% endhighlight %}

A few quick notes on the example code above. The `attached` function is one of a collection of [component lifecycle functions](http://aurelia.io/hub.html#/doc/article/aurelia/framework/latest/creating-components/3) provided by Aurelia. One piece of this puzzle that tripped me up for a while was that toastr is manipulating the DOM through adding a container component and then adding the toasts to that container. For this to work properly, the library's action must be taken sometime after(or during) the `attached` portion of the components lifecycle as that is where the component is attached to the DOM. For a while I was trying to show a toast during bind, which wouldn't due to the previous reason. The click event is something that would be initiated by the user well after the component has been attached to the DOM.

Hope this helps others avoid some of the pitfalls I encountered and gets them up and running with these amazing tools a little more effortlessly.

----
*A few things of note regarding the css files. In the above example, I am loading the css file as a part of the html template utilizing a `<require>` element. The question has been asked regarding utilizing an `import` syntax in the .ts file to load the css instead. There is an [issue](https://github.com/aurelia/cli/issues/273) that has been created on the aurelia-cli repository regarding this. Basically, the dependency loader that is used by the aurelia-cli doesn't support that as of yet. In addition to that, there is a [question on Stack Overflow](http://stackoverflow.com/questions/39355900/css-managment-with-the-aurelia-cli-every-view-loads-another-css-file-to-be-enfo) that raised an issue that I wasn't previously aware of regarding both of these methods of css loading. Basically both of these methods take the css file that is being imported/required and injects it into the head of the document. The problem is that it doesn't remove that css file when one navigates to a different view. This can cause unexpected styling changes to one's application. For a library like select2, that might not be an issue because they create pretty unique element selectors that aren't likely to come into conflict with other pieces of one's HTML. Since that is the way it works though, I find it cleaner and clearer to add the `<require>` element to my `app.html`. This to me, helps to more clearly demark those css files as global files helping to remove some of the confusion.*

----
