---
layout: post
permalink: /frontend/frameworks/aurelia/select-2-with-typescript
title: Aurelia and Select2 using Typescript
excerpt: How to utilize select2 in Aurelia with Typescript typings.
categories: ['Aurelia', 'select2', 'Typescript', 'aurelia-cli']
---

I've utilized [Durandal](http://durandaljs.com/) in the past and while I enjoyed the framework and its power, I kept running into annoyances surrounding the use of [knockout.js](http://knockoutjs.com/). When the Aurelia framework was [announced](http://blog.durandal.io/2015/01/26/introducing-aurelia/), I was excited to give it a try. At that time I had recently started working for a new company that was looking to revamp/rethink the way that they developed software and the tools they utilized to do that. To help us make a decision regarding which front-end framework to use, I took some time to create the same reusable component in [Angular 1.x](https://angularjs.org/), [Angular 2](https://angular.io/), [Aurelia](http://aurelia.io/), [React](https://facebook.github.io/react/) and [Ember](http://emberjs.com/). While I would like to write about that experience, the time to do so hasn't been found yet. In short, we decided that we would like to utilize the power and relative simplicity of the Aurelia framework going forward to create our vNext applications.

The journey so far has been fun, but not without its challenges. I would like to journal some of those challenges and the solutions I've come up with to date.

For an application I was developing, there were a few dropdowns that were required. I wanted a different look and feel for them as I'd experienced before how much of a difference that can make to the overall look and feel of the UI. In the past, I've utilized [select2](https://select2.github.io/) for this purpose. This proved a bit more challenging than I had anticipated and since I've noticed that others have had similar difficulties, I wanted to document it here for the benefit of others.

I first started off following a [blog post](http://ilikekillnerds.com/2015/08/aurelia-custom-element-using-select2-tutorial/) by Dwayne Charrington and a [question](http://stackoverflow.com/questions/33452623/1874522/aurelia-trying-to-load-html-from-select2#answer-33453677) on Stack Overflow. While this was helpful in understanding some of the Aurelia components to the problem, the solution didn't work fully for me. I'm utilizing the Aurelia-CLI and Typescript to build the project which created enough of a difference that I needed to search elsewhere to get up and running. After running into a number of issues, here are the steps I've utilized to get this to work.

After creating a new project utilizing the cli, there are two npm packages that need to be installed.

`npm install jquery select2 --save`

Then the associated typings files need to be installed as well.

`typings install dt~jquery dt~select2 --global --save`

These installs will allow you to utilize these libraries in Typescript, but at the moment, things will fail transpilation if you run `au build`. The reason for this is that there is a variable name declared in the angular-protractor index.d.ts file that conflicts with a declaration of the same name in jquery's .d.ts file. In order to fix this, I needed to comment out line 1839(`declare var $: cssSelectorHelper`) in `typings/globals/angular-protractor/index.d.ts`.

Following this, you need to add two entries in the `aurelia_project/aurelia.json` file. Anywhere in the `bundles.dependencies` section as follows

{% highlight json linenos=table %}
"jquery",
{
  "name": "select2",
  "path": "../node_modules/select2/dist",
  "main": "js/select2.min",
  "resources": [
    "css/select2.min.css"
  ]
}
{% endhighlight %}

Once all of this is done, you are now ready to create your custom attribute. The contents of the .ts file should look as follows:

{% highlight typescript linenos=table %}
import {customAttribute, DOM, inject} from 'aurelia-framework';
import 'jquery';
import 'select2';

@customAttribute('select2')
@inject(DOM.Element)
export class Select2CustomAttribute {
  private value: any;

  constructor(private element) {
  }

  public attached() {
      $(this.element).select2(this.value).on('change', evt => {
          if (evt.originalEvent) {
            return;
          }

          this.element.dispatchEvent(new Event('change'));
    });
  }

  public detached() {
    $(this.element).select2('destroy');
  }
}
{% endhighlight %}

Then you should be able to use your new custom attribute in an `foo.html` file as follows.
{% highlight html linenos=table %}
<template>
    <require from="select2/css/select2.min.css"></require>
    <require from="../custom-attributes/select2"></require>

    <label for="single-select2-select">Single Select2 Select
      <select
          id="single-select2-select"
          select2.bind="selectOptions"
          value.bind="selectedValue"
          change.delegate="changeCallback($event)">
        <option repeat.for="val of singleSelectValues" model.bind="val">
          ${val}
        </option>
      </select>
    </label>
    <label for="multiple-select2-select">Multiple Select2 Select
      <select 
        id="multiple-select2-select"
        select2
        multiple
        value.bind="selectedValues"
        change.delegate="changeCallback($event)">
        <option repeat.for="val of multipleSelectValues" model.bind="val">
          ${val}
        </option>
      </select>
    </label>
</template>
{% endhighlight %}
With the view model(`foo.ts`) looking something like the following:

{% highlight typescript linenos=table %}
export class Foo {
  selectOptions = { allowClear: true, placeholder: 'Select' };
  selectedValue: string = '';
  singleSelectValues: string[] = ['a', 'b', 'c'];
  selectedValues: string[] = [];
  multipleSelectValues: string[] = ['z', 'y', 'x'];

  /* Justification: this is a recommended fix for an issue with Select2 and Aurelia integration as documented here http://stackoverflow.com/questions/33452623/aurelia-trying-to-load-html-from-select2#answer-34121891 */
  /* tslint:disable-next-line no-empty */
  changeCallback(evt: Event): void {
  }
}
{% endhighlight %}

A few things of note regarding the css files. In the above example, I am loading the css file as a part of the html template utilizing a `<require>` element. The question has been asked regarding utilizing an `import` syntax in the .ts file to load the css instead. There is an [issue](https://github.com/aurelia/cli/issues/273) that has been created on the aurelia-cli repository regarding this. Basically, the dependency loader that is used by the aurelia-cli doesn't support that as of yet. In addition to that, there is a [question on Stack Overflow](http://stackoverflow.com/questions/39355900/css-managment-with-the-aurelia-cli-every-view-loads-another-css-file-to-be-enfo) that raised an issue that I wasn't previously aware of regarding both of these methods of css loading. Basically both of these methods take the css file that is being imported/required and injects it into the head of the document. The problem is that it doesn't remove that css file when one navigates to a different view. This can cause unexpected styling changes to one's application. For a library like select2, that might not be an issue because they create pretty unique element selectors that aren't likely to come into conflict with other pieces of one's HTML. Since that is the way it works though, I find it cleaner and clearer to add the `<require>` element to my `app.html`. This to me, helps to more clearly demark those css files as global files helping to remove some of the confusion.

Hope this helps others avoid some of the pitfalls I encountered and gets them up and running with these amazing tools a little more effortlessly.
