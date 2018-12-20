---
layout: post
permalink: /visual-studio-code/debug-angular-build
title: Debugging an Angular build with VSCode
excerpt: How to debug an Angular build in VSCode.
categories: ['Visual Studio Code', 'Angular', 'debugging']
---

I don't know about you dear reader, but to me, almost all code that I haven't written is magical. What do I mean by that? A lot of times I can have thoughts about how a given result was achieved, so its not like the classic 'magicians never share their secrets' magic. Instead, I don't have to worry about the actual implementation, it "Just Works". That is, of course, until it doesn't.

This happened to me recently, trying to create a library for Angular.

----
FWIW: The Backstory

I'm working on a shared Angular library for an organization. One of the components is using a CSS grid layout to achieve a given look. I created an application that I could use to test my components during development and alongside it, the library with the component. [I noticed](https://stackoverflow.com/questions/51600295/how-can-autoprefixer-work-in-angular-with-single-file-components) that while running the application, if it was a multi-file component, I would get the browser prefixes I would expect(in this case `grid { display: grid; }` would transform to `grid { display: -ms-grid; display: grid; }`, but if it was a single file component, I would only get `grid { display: grid; }`. So as long as I created multi-file components, problem solved, right?

Wrong. :(

As it turns out, if I kept to the multi-file components, everything would work with my test application. When I built and published the library though, I was experiencing the same issue where my component wasn't laid out properly in IE because the additional `-ms-grid` property wasn't there. It turns out that the build process for a library takes the multi-file components and creates a single-file component out of it. *Please refer [here](https://github.com/angular/angular-cli/issues/11480#issuecomment-403082982) if you want to know why this is the case.* This is fine and dandy, except it didn't seem to be applying the vendor prefixes when it did so.

----

I attempted to ask a [question](https://stackoverflow.com/questions/51968945/how-do-i-need-to-configure-ng-packagr-to-apply-autoprefixer) on Stack Overflow and made some comments on [an issue](https://github.com/angular/angular-cli/issues/11480) in the Angular cli GitHub repo to no avail. I was getting so frustrated, that I decided I would try to debug through the build process to see if I could determine the source of the error. I love VSCode and have been developing in it for a while, but haven't ever really tried to debug my code too much through it. I've done that in Visual Studio for c# code and in the browser for front-end code(and a lot of help from sourcemaps ;)).

This was a new experience for me and was a lot more difficult than I was expecting. Things make a bit of sense now, but I thought I would share the learning process to remember the solution and hopefully aid others in not stumbling as much as I did when trying to achieve the same result.

Going to the debug icon, it offered for me to create a new configuration as I didn't currently have any in my project. The most logical choice for me was to debug an NPM task as that is how I normally run my builds. Choosing this option produced JSON looking like this:

{% highlight json linenos=table %}
{
  "type": "node",
  "request": "launch",
  "name": "Launch via NPM",
  "runtimeExecutable": "npm",
  "runtimeArgs": [
    "run-script",
    "debug"
  ],
  "port": 9229
}
{% endhighlight %}

This seemed like a nice start, but running it gave me some weird error. I spent a bunch of time flailing at this step, trying to add other (what seemed like logical) properties to the config, all without change. Searching for 'vscode debug npm' led me to the [official documentation](https://code.visualstudio.com/docs/nodejs/nodejs-debugging#_launch-configuration-support-for-npm-and-other-tools) which didn't help much, mostly because I didn't read it so closely (even when I did later, it still seems to be missing a step). I went a bit further down and saw the auto detect debug stuff, which seemed cool, but also didn't work for me. At that point, I came across this [SO question](https://stackoverflow.com/questions/34835082/how-to-debug-using-npm-run-scripts-from-vscode#comment-69464579)(and more importantly the comment).

I was finally getting different error messages when I tried to attach the debugger, which led me to believe I was getting closer. The script I was trying to run/debug originally looked like this: `ng build pd-ng`. After everything I had read and tried though, I managed to get it to this: `node --nolazy --inspect-brk=9229 node_modules/.bin/ng build pd-ng`. I kept getting weird errors and while the error changed if I used `ng` vs `ng.cmd` it wasn't really making a difference. Throughout this process, at various points, some of the errors led me to modify the launch script json to include `outFiles`(and a few others I can't remember) because it seemingly couldn't find js files.

At some point it dawned on me to look into the actual ng/ng.cmd files I was trying to launch which looked as follows:

{% highlight cmd linenos=table %}
@IF EXIST "%~dp0\node.exe" (
  "%~dp0\node.exe"  "%~dp0\..\@angular\cli\bin\ng" %*
) ELSE (
  @SETLOCAL
  @SET PATHEXT=%PATHEXT:;.JS;=;%
  node  "%~dp0\..\@angular\cli\bin\ng" %*
)
{% endhighlight %}

As can be seen, this is launching node, pointing it to execute one of Angular's scripts. Armed with that knowledge, I was able to get my debug script running. Here is the final result:

`.vscode/launch.json`

{% highlight json linenos=table %}
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
      {
        "type": "node",
        "request": "launch",
        "name": "Launch via NPM",
        "cwd": "${workspaceFolder}",
        "runtimeExecutable": "npm",
        "windows": {
          "runtimeExecutable": "npm.cmd"
        },
        "runtimeArgs": [
          "run-script",
          "build:debug"
        ],
        "port": 9229
      }
    ]
}
{% endhighlight %}

`package.json scripts`

{% highlight json linenos=table %}
"build:debug": "node --nolazy --inspect-brk=9229 node_modules/@angular/cli/bin/ng build pd-ng"
{% endhighlight %}

After all of this work, I wish that I could tell you I found the source of the bug. Sadly, I haven't, but just being able to debug and learning a bit more about VSCode has been a great experience in its own right.
