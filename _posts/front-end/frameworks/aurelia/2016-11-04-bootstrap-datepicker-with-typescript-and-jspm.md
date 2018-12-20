---
layout: post
permalink: /frontend/frameworks/aurelia/bootstrap-datepicker-with-typescript-and-jspm
title: Aurelia and Bootstrap Datepicker using Typescript and jspm
excerpt: How to utilize bootstrap datepicker in Aurelia with Typescript typings and jspm.
categories: ['Aurelia', 'jspm', 'Typescript', 'bootstrap-datepicker']
---

I find that developing front-end applications utilizing the Aurelia framework is really a pleasure. The one thing that I struggle with though is the popularity of Angular 2.0, the ecosystem around it and the lack that seems to be there for Aurelia.

Developing in the modern world of Javascript can be quite challenging. To effectively work on a front-end application, I find the need to be familiar with npm, jspm, html, css, javascript(multiple different flavors: es5, emerging specs like es6, es7, etc.), gulp, aurelia, typescript and typings. While I realize that some of these are optional or replaceable with other similar libraries, the idea is still the same. One needs to be able to know a variety of languages and their nuances as well as all the supporting technology. The languages are constantly evolving as well as the supporting ecosystems, not least of which is the variability regarding browser implementations.

While this is true of any healthy language, it seems to be particularly fast-paced in the world of javascript. I believe some of the issues could be resolved with better tooling around all of these libraries and have seen a nice push forward with that using Visual Studio Code. Another area of improvement should come in the area of documentation. This particular problem seems to be quite significant in the javascript ecosystem. There are tons of small modules, many of which do similar things. How does one know what is out there and how to choose between the various options.

While I feel as though Aurelia does a pretty decent job with documentation, I realize that it is a hard problem and am unsure how to suggest a way forward. As I struggle getting third-party libraries working with Aurelia or fight through other implementation issues, I first try to search for answers. I then look to StackOverflow. If all else fails, I try to fight through it by trying to make educated guesses about the issue and solution. I have tried the Gitter channel which by and large hasn't been too helpful and if I need to, I give up and try to find another tool that might solve the problem and the cycle starts all over again.

Why is it so hard? They are trying to provide guidance and documentation on how to utilize the framework with a variety of different back ends and multiple different package managers, build tools and languages(really varieties of the same language, but which can be quite distinct at times) on the front end. There are a number of good places to start, but I often find that they are utilizing a bit of a different stack which doesn't fully get me to where I need to go.

I would like to document here the solutions I've come up with in the hope of adding to Aurelia's ecosystem and potentially providing solutions for others so that they won't need to struggle through the same things I have for as long.

I've had a need in multiple applications to provide the user with a datepicker. While creating `<input type="date" />` in a page that is displayed in Chrome will automatically provide the user with a nice datepicker, unfortunately that doesn't exist in all of the browsers. There is a third-party library that seems to handle all browsers pretty nicely and that is [bootstrap-datepicker](https://github.com/uxsolutions/bootstrap-datepicker).

After working on it for a while, here are the steps I took to get it working with the Aurelia framework:

I started with the [skeleton-typescript](https://github.com/aurelia/skeleton-navigation/releases/). At the time of writing, the latest version is 1.0.3. This utilizes gulp for the front end task runner, jspm as the UI package manager/module loader as well as npm for package management relating to tasks run by the task runner and typescript as the front-end development language.

----
**NOTE**: One issue I've run into utilizing typescript to work on Aurelia applications is that anytime I need to utilize jQuery to get a third-party library to function correctly, there is a typings conflict with the angular-protractor typings file(they both export `$`). At the moment, I'm not creating any e2e tests for my applications, so my solution is to remove two lines from the `typings.json` file, namely, "angular-protractor" and "aurelia-protractor" from the "globalDevDependencies" section. I would like to find a better long-term solution, but at the moment, this is sufficient for my needs.

----

The Aurelia website says you have to run three commands to get all of the dependencies installed for your project: `npm install`, `jspm install`, `typings install`. Really if you look at the `package.json` file though, under the scripts sections, you will notice one that looks like this `"prepublish": "./node_modules/.bin/typings install"`. If you add another one under it with this: `"postinstall": "./node_modules/.bin/jspm install"`, then you will only have to run the `npm install` command.

As a precaution, I usually run gulp watch after this so I can make sure everything installed properly and the project runs before I start my updates.

Now, for the bootstrap-datepicker specifics:

There are a number of commands that need to be run from the command line to get the necessary packages/typing files.

```
jspm install npm:bootstrap-datepicker

typings install dt~bootstrap-datepicker --save --global

typings install dt~jquery --save --global
```

----
**NOTE**: You don't need to run `jspm install jquery` because that is included in the skeleton.

----

Create a file for your "custom attribute"(I called mine `datepicker.ts` and placed it in `src/custom-attributes`) with the following contents:

{% highlight typescript linenos=table %}
import {DOM, customAttribute, inject} from 'aurelia-framework';
import 'bootstrap-datepicker';

@customAttribute('datepicker')
@inject(DOM.Element)
export class DatepickerCustomAttribute {
    private value: Date;

    constructor(private element: Element) {
    }

    public attached() {
        let datepickerOptions: DatepickerOptions = { autoclose: true, format: 'yyyy-mm-dd' };

        $(this.element)
            .datepicker(datepickerOptions)
            .on('changeDate', evt => {
                this.value = evt.date;
            });
    }

    public detached() {
        $(this.element).datepicker('destroy');
    }
}
{% endhighlight %}

----
**NOTE**: If you didn't remove the lines in `typings.json` file relating to the 'protractor' typings files, you will get an error with the above code. Again, this is do to the conflicting `$` exports between 'angular-protractor' and 'jquery' typings files. To fix this, you can update your `typings.json` file and then delete the directory at `typings/globals/angular-protractor` or you can simply comment out line 1839 `declare var $: cssSelectorHelper;` at the file located `typings/globals/angular-protractor/index.d.ts`.

----

Just to provide a fully working example, I updated `welcome.html` to the following:

{% highlight html linenos=table %}
<template>
  <require from="./custom-attributes/datepicker"></require>

  <div datepicker.two-way="date"></div>
</template>
{% endhighlight %}

**UPDATE**
*As noted by Jerry T(THANKS!) in a comment, the datepicker doesn't hide and show by default the way one would expect if the html is as stated above, with the binding being on a div element, but the datepicker still allows a user to pick a date and the binding correctly changes. For the more full set of functionality to work, utilize the `welcome.html` below.*

{% highlight html linenos=table %}
<template>
  <require from="./custom-attributes/datepicker"></require>

  <input datepicker.two-way="date" type="date" />
</template>
{% endhighlight %}

And `welcome.ts to` the following:

{% highlight typescript linenos=table %}
import {bindable} from 'aurelia-framework';

export class Welcome {
  @bindable public date = null;

  public dateChanged(newValue, oldValue) {
    console.log('new:' + newValue);
    console.log('old:' + oldValue);
  }
}
{% endhighlight %}

----
**NOTE**: This should work as is. I wanted to provide a fully functioning example. I have utilized other third-party tools as well in other components. One of the one's I used was select2. When I got it all setup and working, both components didn't seem to be able to render on the same page. Digging into it a bit more, it seems to be a problem with different jquery versions being required between the two libraries. I have yet to find a satisfactory long-term solution to this. At the moment, the way that I get this to work is through deleting a few lines from the `config.js` file after executing `jspm install`. In the "map" section, I looked for bootstrap-datepicker and select2 and removed the lines containing their jquery dependencies. These get put back everytime one runs `jspm install`, so as I noted, it isn't a long term solution, but it does work for now.

----

Hope this helps!
