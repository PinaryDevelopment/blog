---
layout: post
permalink: /front-end/npm/scripts/getting-started
title: "npm scripts: Getting Started"
excerpt: How to get started with npm scripts.
categories: ['npm', 'npm-scripts', 'Node.js', 'JavaScript']
---

Around the time I started focusing my career more on web development, there were a few build engines emerging to help with the build tasks relating to front-end development files. They were [grunt](https://gruntjs.com/) and [gulp](https://gulpjs.com/). Most of the discussion at the time was around code vs. configuration and gulp's usage of streams, as opposed to writing to disk for each operation, to speed up their builds. I don't remember anyone mentioning [npm-scripts](https://docs.npmjs.com/misc/scripts) at the time, nor quite frankly, much in the mean time. Most front-end starter kits include some npm scripts in their starter kits and in learning about them a little more, I've found them quite useful for many tasks. React, Angular and Aurelia's starter projects all include at least one npm script. Angular seems to use gulp in their build scripts whereas React seems to utilize npm scripts alone and Aurelia appears to use a combination of the two.

At some point, I realized that I could create my own npm-scripts and began to play around with them to start to define where I could gain the most value from them in my day to day development. I found out a number of very useful things along the way. While I've no doubt just started to scratch the surface of this utility, I thought I would share some of my initial discoveries.

By way of introduction, any application that utilizes npm to manage its packages has a `package.json` file that contains many properties. Some include the project's name, (optionally) authors, license and dependencies. One of the properties(and the focus of this post) is the `scripts` property. This is an area where you can declare scripts that can call executables by npm for the given project. You give each script a name and then tell it what scripts to execute.

The easiest way to see this is through the following steps:
{% highlight cmd linenos=table %}
mkdir npm-scripts-demo
cd npm-scripts-demo
npm init *(then accept all of the defaults)*
{% endhighlight %}

Once complete, the `package.json` that gets created has the following script: `"test": "echo \"Error: no test specified\" && exit 1"`. From the command line, you can run `npm run test` and you should see the script get executed. It will first echo out `"Error: no test specified"` and since the script then exits with a value of 1, it will also give the user an error message.

As a bit of an aside, but shown in the `test` script above, with npm scripts, one can run multiple commands in a single script. If the commands are separated by `&&` they will run sequentially and if they are separated by `|` or `&`, depending on the execution environment, they are run concurrently. *(NOTE: see [this StackOverflow Q&A](https://stackoverflow.com/questions/30950032/how-can-i-run-multiple-npm-scripts-in-parallel#answer-30950298) for a more cross-platform way of doing this)* This can be tested out by updating the above script to `"test": "echo \"Error: no test specified\" > test.txt | exit 1"`. As can be seen when running the script, the error message shows in the console, but the file gets created with the expected text as well.

Now the above example is pretty trite, but provides a good starting point. `echo` is a global command exposed in most operating systems through their command line. Executing npm scripts will mostly be used to execute commands from installed npm packages. As an example, a common operation to be performed during local development will be to delete a directory containing compiled artifacts before compiling them anew following code changes. This might be `.css` files that were composed from `.scss` files or `.js` files transpiled from `.ts` files. A common library used to perform this task is `rimraf`.

To demonstrate: Install `rimraf` through the command `npm install rimraf`(this won't save it to your `package.json` file, but will install the package in the `node_modules` directory relative to where the install command was executed). To utilize `rimraf` to delete the test file created above run the command `node_modules/.bin/rimraf test.txt` in the command line. As can be seen, when npm installs an executable script, it places it in the `node_modules/.bin` directory. If we wanted to execute that command as an npm script called `clean`, we would update the scripts property to include the following `"clean": "rimraf test.txt"` and then we could execute that from the command line with `npm run clean`. Now I'm sure you're wondering, Why isn't the task declaration exactly like the command line statement we executed above? In truth, we could have written the script as such: `node_modules/.bin/rimraf test.txt`, but npm scripts places the `node_modules/.bin` directory in its path, abrogating the need for the more verbose command. I prefer the shorter command as it keeps the scripts section a bit more readable.

There are multiple ways that one can define the order in which scripts run. One method was described above utilizing the `&&` operator. There is another, which relies on an npm convention that I've found useful as well. By default, if npm is told to run a script(i.e. `npm run test`), it will look for a task called `pretest` and if it finds one, it will run that script, then the `test` script. Upon completion of the `test` script, it will look for a script with the name `posttest` and if it finds one, it will run that last. This can be demonstrated by adding: `"pretest": "echo \"Hello World!\""` to the list of scripts and then executing `npm run test`. This can be done for any task. Defining a task with the same name prefaced by `pre` or `post` will have the same execution effect. I have even used this functionality to inject some file changes to a node_modules package after it got installed through declaring a `postinstall` task.

All of the above is really great and has really helped me a lot. To take it one step further: There are often times where running a cli command isn't enough for the task at hand. What happens when we need to create more complex scripts and want to execute them using npm scripts? We can create a file that contains all of the code that needs to run and execute that from npm scripts. We can rewrite our `clean` method above to demonstrate.

Create a `clean.js` file with the following contents:

{% highlight javascript linenos=table %}
var rimraf = require('rimraf');

function onError(err) {
    console.log(err);
}

rimraf('test.txt', onError);
{% endhighlight %}

Update the clean task to: `node clean`

Now when you run the `npm run clean` script, it will instruct node to execute the `clean.js` file as a script. This loads and runs `rimraf` with the arguments provided.

While this is a very simple and not very useful example, it demonstrates the capability. It obviously can be built upon to perform much more complicated actions. I've used this before to compile .scss files with sourcemaps, add a hash to the filename and copy them to the websites root directory.

Hope this helps getting started with npm and being more productive with it from day 1!
