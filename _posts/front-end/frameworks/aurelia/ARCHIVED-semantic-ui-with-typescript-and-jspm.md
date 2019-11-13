---
layout: post
permalink: /frontend/frameworks/aurelia/semantic-ui-with-typescript-and-jspm
title: Aurelia and Semantic UI using Typescript and jspm
excerpt: How to utilize Semantic UI in Aurelia with Typescript typings and jspm.
categories: ['Aurelia', 'jspm', 'Typescript', 'Semantic UI']
---

There was [a question](http://stackoverflow.com/questions/41052178/replace-bootstrap-with-semantic-ui-for-aurelia-starter-kit/41052897) raised on Stack Overflow the other day. The questioner wanted to know how they could use [Semantic UI](http://semantic-ui.com/) in place of [Bootstrap](http://getbootstrap.com/) as their UI component library. Bootstrap comes with the started kits and semantic ui doesn't. Getting started with these frameworks can be quite challenging, especially when it comes to integrating outside libraries. I almost passed up the question as I've never used Semantic UI before, but soon realized that the question had almost nothing to do with that library specifically, but rather its integration into Aurelia. This is something I've had to do a few times and thought I would give it a shot.

I posted [an answer](http://stackoverflow.com/questions/41052178/replace-bootstrap-with-semantic-ui-for-aurelia-starter-kit/41052897#41052897) there, but wanted to try to give it a bit more of a full write up here.

As stated there, to solely switch out the packages and the base css file loaded, here are the steps that need to be taken:

######Remove bootstrap
*Option 1*: Manually remove the bootstrap entries from the `config.js` and the `package.json` files.

*Option 2*: Run the command `jspm uninstall bootstrap`.

######Install Semantic UI
Run the command `jspm install semantic-ui`.

There are two other changes that need to made.

In `app.html`
change `<require from="bootstrap/css/bootstrap.css"></require>`
to `<require from="semantic-ui/semantic.css"></require>`.

In `main.ts`,
change `import 'bootstrap';`
to `import 'semantic-ui';`.

After making those changes, run `gulp watch` and when you launch the browser and load the application, if you have your dev tools open and are looking at the network tab, you should be able to see the `semantic.js` and `semantic.css` files getting loaded by the browser in place of the bootstrap files. The page doesn't look nicely styled though as we didn't update any of the html elements to reflect semantic-ui's classes or element structure.

There are two other changes that should be made for the build/bundle gulp tasks as well.

In `build/bundles.js`,
change
`"bootstrap",
"bootstrap/css/bootstrap.css!text",
`
to
`"semantic-ui",
"semantic-ui/semantic.css!text",`

In `build/export.js`
change
`'bootstrap', [
  '/fonts/*'
]
`
to
`'semantic-ui', [
  '/themes/{THEME_NAME_OF_CHOICE}/*'
]`

That was the end of my answer there. I've been meaning to write this post for a while though. One of the other answers to the question points to a blog post that discussed the creation of a custom element that taps into the power of semantic-ui's jQuery library as well. The user 'Matthew James Davis' there goes through the steps of creating an Aurelia custom element that wraps one of semantic ui's jQuery elements.

`typings install semantic-ui=https://raw.githubusercontent.com/budiadiono/semantic-ui-typescript/master/output/semantic-ui.d.ts --global --save`

`typings install dt~jquery --global --save`

If that is what you are looking for, please be more specific with your question.

Hope this helps!

http://davismj.me/blog/semantic-custom-element/
https://github.com/Semantic-Org/Semantic-UI/issues/3091




**TODO ts**

import { inject, bindable, customElement, bindingMode } from 'aurelia-framework';
import 'semantic-ui';

@inject(Element)
@customElement('semantic-select') 
export class SProgressCustomElement {

  @bindable({ defaultBindingMode: bindingMode.oneTime }) label;
  @bindable({ defaultBindingMode: bindingMode.oneWay }) labeled;
  @bindable progress;

  constructor(private element: Element) {
  }

  bind() {
    this.labeled = this.labeled || this.element.hasAttribute('labeled');
    this.label = this.label || this.element.getAttribute('label');
    this.progress = this.progress || this.element.getAttribute('progress');
  }

  attached() {
    $(this.element).progress({
      text: {
        active: this.label
      }
    });
  }

  progressChanged(newValue) {
    $(this.element).progress('set progress', newValue);
  }
}

**TODO html**
<template>
    <select class="ui fluid dropdown" value.bind="selectedOption"> <!--multiple="multiple"-->
        <option repeat.for="option of options" model.bind="option">${option}</option>
    </select>
</template>
