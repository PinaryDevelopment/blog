---
layout: post
permalink: /frontend/frameworks/aurelia/bootstrap-datepicker-with-typescript
title: Aurelia and Bootstrap Datepicker using Typescript
excerpt: "How to utilize bootstrap datepicker in Aurelia with Typescript typings and Aurelia's cli."
categories: ['Aurelia', 'aurelia-cli', 'Typescript', 'bootstrap-datepicker']
---

This is a bit of a long post, but it is one I squeezed in due to a very interesting back and forth I had with 'Jerry T' the other day in the comments to my blog post entitled [Aurelia and bootstrap-datepicker using typescript and jspm](/aurelia-and-bootstrap-datepicker-using-typescript-and-jspm). Two main themes were touched upon: Aurelia vs. Angular(which I hope to write about in a later post), Aurelia-cli vs Aurelia's skeleton projects.

One of the challenges that comes with doing front-end development these days revolves around the breakneck speed of change. Even if you have settled on a framework, they are constantly improving the frameworks and creating releases for those improvements. That is ***AWESOME***, but at the same time can be **VERY** challenging to deal with. Documentation that was there one day, might be irrelevant or completely gone the next day. Trying to figure out what information(blogs, StackOverflow Q&As) is relevant to the specific version you are working with can be extremely difficult and knowing when and how to update these libraries in order to not fall too far behind is also a balancing act.

For my day job, the CTO of the company wanted to use Angular\*new\* for their projects. I have utilized and recommended Aurelia in the past and have tried to use it for a personal project that I'm in the middle of, in order to keep current with it. There are a number of things I really like and dislike about both Aurelia and Angular.

As of this writing, the above post was created 8 months ago. A lot has changed since then. First, I noticed that Jerry was utilizing the Aurelia skeleton projects as his starting point. To be honest, 8 months ago, that's what I was using as well. The cli was pretty new, raw and frustrating to use. Looking at it now though, there hasn't been a new release of the skeletons for ~7 months. At first glance, it looked like the repository for the skeleton projects was active. I was about to 'pen' a comment recommending that if one wanted to use the skeleton projects, they should probably clone the repo and start from there. On further inspection though, while still showing some signs of activity, the repository doesn't seem to have had any meaningful changes made to it in months as well.

On the other hand, the cli had a release a week ago. The official Aurelia documentation gives both the cli and the skeletons space, but it seems as though the cli is the framework's preferred choice for success. I remember initially that one of the biggest hurdles I had with the cli was integrating third-party libraries. While the documentation was there, it never seemed to 'just work' the way I would have expected. I recommended that Jerry give the cli a try and he suggested I write a post about the cli ;). While this isn't that post exactly, I decided to revisit third-party library integration and the cli and I figured, what better way than to provide a part II to the bootstrap-datepicker integration post.

As it turns out, much to my surprise, this was relatively easy. I'll spell out the steps I took, annoyances encountered and some thoughts on the cli along the way.

To start, I created a new folder on my desktop `au-cli`. Obviously, this can be whatever you want it to be, but that name was sufficient for me.

>***This is my opinion and many would disagree with me***: You can install the cli globally, but definitely given the rate of change, I like to install the cli locally(`npm install aurelia-cli`) every time I start a new project so I know I'm getting the latest release. Since I don't install it globally, a lot of my npm commands will look like `node_modules/.bin/au ...`. If you have the cli installed globally(`npm install -g aurelia-cli`, you should run these commands as `au ...` instead.

Given the above note, in the `au-cli` directory, I opened my command prompt(git bash shell actually) and ran `npm install aurelia-cli`. Again, you can skip this if you have it installed globally.

Then to create a new project execute: `node_modules/.bin/au new third-party-test`. I called it third-party-test, but again, call the project whatever suits you best. This will give you a list of options you must choose from for it to configure the project properly. Here are the options I chose:

| Loader? | 1. RequireJS |
| Default or custom setup? | 3. Custom |
| Transpiler? | 2. Typescript |
| Template setup? | 3. Maximum minification |
| Css Preprocessor? | 3. Sass |
| Configure unit testing? | 1. Yes |
| Default code editor? | 1. VSCode |
| Create project? | 1. Yes |
| Install dependencies? | 1. Yes |

Once it is done with the install, this was the first annoyance I encountered. The command prompt looked as follows:
![](/assets/images/au-cli_new_end.png)

It has a nice message and the 'Happy Coding!' at the end would seem to indicate that the setup is done and it is time to get down to business, but it doesn't exit and display the cursor. I waited a while, expecting it to do just that when it was completely done, but it never did. Instead, you have to press `Ctrl + c` to get your cursor back.

Then I navigated into the project directory `cd third-party-test`. As in the previous post, I tried to run a start command to make sure everything was working and I got an error message. I thought that I didn't have the command correct, so I opened the `project.json` file to see what I should be invoking and found the scripts section missing completely. I assume this is done by design, but why? I would think it would be nice to have a barebones build, run, watch and test command in there to help get started. Since there isn't, I added one to get started:
{% highlight json linenos=table %}
"scripts": {
    "watch": "au run --watch"
  },
{% endhighlight %}

Now I ran `npm run watch`, launched a browser and navigated to `http://localhost:9000`. Assuming everything works you should see the 'App works!' message on the screen. While the skeletons come with a few example files, the cli assumes you know what you are doing and only comes with the bare minimum Aurelia project. In a similar vein, it doesn't come with Bootstrap and jQuery integrated by default.

At this point it is time to add the third-party libraries. Looking at the official documentation, the cli seems to be [fairly well documented](http://aurelia.io/hub.html#/doc/article/aurelia/framework/latest/the-aurelia-cli)(though it seems to be out of date). Looking through it, there is a section dedicated to [Adding Client Libraries to Your Project](http://aurelia.io/hub.html#/doc/article/aurelia/framework/latest/the-aurelia-cli/10). Before I followed that blindly though, I ran `node_modules/.bin/au help` in my command prompt. Low and behold I saw this in the console:
![](/assets/images/au-cli_help.png)

Before just trying out the highlighted commands, I tried to find the documentation for those commands on Aurelia's site and much to my surprise, found ***NONE***. I decided to give the cli version a try anyway.
Running the command `node_modules/.bin/au install bootstrap-datepicker jquery` installs those packages and updates the `package.json` file. The console then prompted me to help configure the css files needed that it found in the installed packages. At this time I chose option 2 which didn't configure any of the css files as my original post didn't include them either. *At a later step I do add one of them in.* Again, at this point, the cli doesn't exit and return the cursor(now I'm a little peeved as this seems by design, not just an oversight and I'm not sure what the benefit of this 'feature' is).

Now for the code changes:
I created the file `src/resources/attributes/datepicker.ts` with the contents:

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

Updated `src/resources/index.ts` to:

{% highlight typescript linenos=table %}
import {FrameworkConfiguration} from 'aurelia-framework';

export function configure(config: FrameworkConfiguration) {
  config.globalResources(['./attributes/datepicker']);
}
{% endhighlight %}

Updated `src/app.ts` to:

{% highlight typescript linenos=table %}
import {bindable} from 'aurelia-framework';

export class App {
  @bindable public date = null;

  public dateChanged(newValue, oldValue) {
    console.log('new:' + newValue);
    console.log('old:' + oldValue);
  }
}
{% endhighlight %}

Updated `src/resources/index.html` to:

{% highlight html linenos=table %}
<template>
  <input datepicker.two-way="date" type="date" />
</template>
{% endhighlight %}

Now when I ran `npm run watch` again, there are a few errors thrown by gulp, but the project continues to successfully build(is this by design? if so, why? if there are build errors, shouldn't execution stop?) and launch. Navigate to `http://localhost:9000` and you can see the datepicker showing in the UI. Pretty easy and cool!

I wanted to get the build errors to go away which was easy. In the install step before, I neglected to include the typings files. The old blog post talks about running `typings install...`, but npm and the typescript/typings world have evolved as well and the typings files can be installed with npm now. Run `npm install --save-dev @types/jquery @types/bootstrap-datepicker` and when you run the project again, you will see that the build errors are now gone.

One more thing I thought I should add for completeness was one of the datepicker stylesheets. As is, the datepicker appears, but it doesn't look very good. As seen above, the cli has an `install` command and an `import` command. I believe the intent of the import is for when one installs a dependency through npm and then realizes that it needs to be hooked into the Aurelia build. This could be done as the documentation states by updating the `aurelia_project/aurelia.json` file by hand, but I ran `node_modules/.bin/au import bootstrap-datepicker` and it provided me with the same prompt as before to help include the css files in the bootstrap-datepicker npm package in this project. This time I chose 1.(I want choose which css files I need). It then displays the css files it found in that package, I chose to include 2(dist/css/bootstrap-datepicker.min.css). I ran the application again and, *sad-trumpet*, the datepicker doesn't appear any differently. When I used the developer tools, I can see that the css file is included in the `vendor-bundle.js` file, but the styles aren't displaying. In order to get them to show up, I had to add the requisite `require` tag to the application. I updated the `src/app.html` to this:

{% highlight html linenos=table %}
<template>
  <require from="bootstrap-datepicker/dist/css/bootstrap-datepicker.min.css"></require>
  <input datepicker.two-way="date" type="date" />
</template>
{% endhighlight %}

Since watch was running, when I saved it, the app recompiled and refreshed the browser and the styles were now applied.

I hope this helps. Happy coding!
