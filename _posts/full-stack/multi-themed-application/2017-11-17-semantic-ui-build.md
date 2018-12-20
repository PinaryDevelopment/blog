---
layout: post
permalink: /fullstack/multi-themed-application/semantic-ui-build
title: "Multi-themed Web Project: Semantic UI One Build for Multiple Themes"
excerpt: Describing problems encountered and the solution for creating multiple themes with the semantic-ui build process.
categories: ['JavaScript', 'Angular', 'Semantic UI', 'Node.js', 'Gulp', 'architecture']
---

While Semantic UI does provide a nice way to create themes, I quickly discovered that its build engine is focused on building one custom theme per build/project. It took me a bit of time and a lot of console logging to come up with a modification to it's gulp build process that would work for building multiple themes in one step. When Semantic UI is installed via npm, whatever directory you specify for your semantic files will contain a `semantic/tasks/build.js` file.

{% highlight javascript linenos=table %}
/*******************************
          Build Task
*******************************/

var
  /* dependencies */
  gulp         = require('gulp-help')(require('gulp')),
  runSequence  = require('run-sequence'),

  /* config */
  config       = require('./config/user'),
  install      = require('./config/project/install'),

  /* task sequence */
  tasks        = []
;


/* sub-tasks */
if(config.rtl) {
  require('./collections/rtl')(gulp);
}
require('./collections/build')(gulp);


module.exports = function(callback) {

  console.info('Building Semantic');

  if( !install.isSetup() ) {
    console.error('Cannot find semantic.json. Run "gulp install" to set-up Semantic');
    return 1;
  }

  /* check for right-to-left (RTL) language */
  if(config.rtl === true || config.rtl === 'Yes') {
    gulp.start('build-rtl');
    return;
  }

  if(config.rtl == 'both') {
    tasks.push('build-rtl');
  }

  tasks.push('build-javascript');
  tasks.push('build-css');
  tasks.push('build-assets');

  runSequence(tasks, callback);
};
{% endhighlight %}

My initial attempt to get the build to work with multiple themes was to modify the above file to the following:

{% highlight javascript linenos=table %}
/*******************************
          Build Task
*******************************/

var
  /* dependencies */
  gulp         = require('gulp-help')(require('gulp')),
  runSequence  = require('run-sequence'),
  print        = require('gulp-print'),
  /* config */
  config       = require('./config/user'),
  install      = require('./config/project/install'),

  /* task sequence */
  tasks        = []
;


/* sub-tasks */
if(config.rtl) {
  require('./collections/rtl')(gulp);
}
require('./collections/build')(gulp);

const orgs = require('../../build/licensed-organizations.json').orgs;
module.exports = function(callback) {
  tasks.push('build-javascript');
  tasks.push('build-assets');
  var lastTaskName = '';

  for(var i = 0; i < orgs.length; i ++) {
    console.info('Building Semantic');
    const org = orgs[i];

    gulp.task(`copy semantic ${org}`, function() {
      console.info(`copy semantic ${org}`);
      return gulp.src(`./orgs/${org}/semantic.json`)
                  .pipe(print())
                  .pipe(gulp.dest('../'));
    });

    gulp.task(`copy theme ${org}`, function() {
      console.info(`copy theme ${org}`);
      return gulp.src(`./orgs/${org}/theme.config`)
                  .pipe(print())
                  .pipe(gulp.dest('./src/'));
    });

    gulp.task(`build css ${org}`, [`build-css`]);

    if( !install.isSetup() ) {
      console.error('Cannot find semantic.json. Run "gulp install" to set-up Semantic');
      return 1;
    }

    tasks.push(`copy semantic ${org}`);
    tasks.push(`copy theme ${org}`);
    tasks.push(`build css ${org}`);
  };

  runSequence(...tasks, callback);
};
{% endhighlight %}

At a high level, it imports the organizations that have been placed in the 'organizations.json' file, iterating over them, creating a uniquely named gulp task for each one and then pushing each task to an array. Once all of the organizations have been iterated over, I'm calling run sequence which should sequentially execute each task and am using the spread operator to pass all of the tasks to that function.

A closer look shows that the tasks attempt to take the `semantic.json` and `theme.config` files created for each organization and overwrites the default files from  semantic and then executes the `build-css` task that the semantic library creators provide to compile all of the less files into the css files that are actually served to the browser.

**The problem with this approach is that the build process only seems to use the original semantic.json file that was in place before the build started even though it is successfully getting overwritten.** For instance, in the original semantic.json file, the value for output.packaged is 'dist/'. semantic.json is successfully getting overwritten and the output.packaged value is dist/org1 before the build-css task gets executed, but all of the output files still end up in 'dist/'.

I decided to dig a bit deeper into the build engine for my second approach. The default build files include a `semantic/tasks/build/css.js` file which contains the tasks relating to the css portion of the Semantic UI build. The file looks as follows:

{% highlight javascript linenos=table %}
/*******************************
          Build Task
*******************************/

var
  gulp         = require('gulp'),

  /* node dependencies  */
  console      = require('better-console'),
  fs           = require('fs'),

  /* gulp dependencies */
  autoprefixer = require('gulp-autoprefixer'),
  chmod        = require('gulp-chmod'),
  clone        = require('gulp-clone'),
  flatten      = require('gulp-flatten'),
  gulpif       = require('gulp-if'),
  less         = require('gulp-less'),
  minifyCSS    = require('gulp-clean-css'),
  plumber      = require('gulp-plumber'),
  print        = require('gulp-print'),
  rename       = require('gulp-rename'),
  replace      = require('gulp-replace'),
  runSequence  = require('run-sequence'),

  /* config */
  config       = require('../config/user'),
  tasks        = require('../config/tasks'),
  install      = require('../config/project/install'),

  /* shorthand */
  globs        = config.globs,
  assets       = config.paths.assets,
  output       = config.paths.output,
  source       = config.paths.source,

  banner       = tasks.banner,
  comments     = tasks.regExp.comments,
  log          = tasks.log,
  settings     = tasks.settings
;

/* add internal tasks (concat release) */
require('../collections/internal')(gulp);

module.exports = function(callback) {

  var
    tasksCompleted = 0,
    maybeCallback  = function() {
      tasksCompleted++;
      if(tasksCompleted === 2) {
        callback();
      }
    },

    stream,
    compressedStream,
    uncompressedStream
  ;

  console.info('Building CSS');

  if( !install.isSetup() ) {
    console.error('Cannot build files. Run "gulp install" to set-up Semantic');
    return;
  }

  /* unified css stream */
  stream = gulp.src(source.definitions + '/**/' + globs.components + '.less')
    .pipe(plumber(settings.plumber.less))
    .pipe(less(settings.less))
    .pipe(autoprefixer(settings.prefix))
    .pipe(replace(comments.variables.in, comments.variables.out))
    .pipe(replace(comments.license.in, comments.license.out))
    .pipe(replace(comments.large.in, comments.large.out))
    .pipe(replace(comments.small.in, comments.small.out))
    .pipe(replace(comments.tiny.in, comments.tiny.out))
    .pipe(flatten())
  ;

  /* two concurrent streams from same source to concat release */
  uncompressedStream = stream.pipe(clone());
  compressedStream   = stream.pipe(clone());

  /* uncompressed component css */
  uncompressedStream
    .pipe(plumber())
    .pipe(replace(assets.source, assets.uncompressed))
    .pipe(gulpif(config.hasPermission, chmod(config.permission)))
    .pipe(gulp.dest(output.uncompressed))
    .pipe(print(log.created))
    .on('end', function() {
      runSequence('package uncompressed css', maybeCallback);
    })
  ;

  /* compressed component css */
  compressedStream = stream
    .pipe(plumber())
    .pipe(clone())
    .pipe(replace(assets.source, assets.compressed))
    .pipe(minifyCSS(settings.minify))
    .pipe(rename(settings.rename.minCSS))
    .pipe(gulpif(config.hasPermission, chmod(config.permission)))
    .pipe(gulp.dest(output.compressed))
    .pipe(print(log.created))
    .on('end', function() {
      runSequence('package compressed css', maybeCallback);
    })
  ;

};
{% endhighlight %}

I updated the file to:

{% highlight javascript linenos=table %}
const console = require('better-console');
const extend = require('extend');
const fs = require('fs');
const gulp = require('gulp');
const autoprefixer = require('gulp-autoprefixer');
const chmod = require('gulp-chmod');
const minifyCSS = require('gulp-clean-css');
const clone = require('gulp-clone');
const concat = require('gulp-concat');
const concatCSS = require('gulp-concat-css');
const dedupe = require('gulp-dedupe');
const flatten = require('gulp-flatten');
const header = require('gulp-header');
const gulpif = require('gulp-if');
const less = require('gulp-less');
const plumber = require('gulp-plumber');
const print = require('gulp-print');
const rename = require('gulp-rename');
const replace = require('gulp-replace');
const uglify = require('gulp-uglify');
const requireDotFile = require('require-dot-file');
const runSequence = require('run-sequence');

const config = require('../config/project/config');
const defaults = require('../config/defaults');
const install = require('../config/project/install');
const tasks = require('../config/tasks');
const banner = tasks.banner;
const comments = tasks.regExp.comments;
const log = tasks.log;
const settings = tasks.settings;
const filenames = tasks.filenames;

const orgs = requireDotFile(`organizations.json`, __dirname).orgs;

module.exports = function(callback) {
    orgs.forEach(org => {
        const userConfig = requireDotFile(`semantic.${org}.json`, __dirname);
        const gulpConfig = (!userConfig) ? extend(true, {}, defaults) : extend(false, {}, defaults, userConfig);
        const compiledConfig = config.addDerivedValues(gulpConfig);
        const globs = compiledConfig.globs;
        const assets = compiledConfig.paths.assets;
        const output = compiledConfig.paths.output;
        const source = compiledConfig.paths.source;

        const cssExt = { extname: `.${org}.css` };
        const minCssExt = { extname: `.${org}.min.css` };

        let tasksCompleted = 0;
        let maybeCallback  = function() {
            tasksCompleted++;
            if(tasksCompleted === 2 * orgs.length) {
                callback();
            }
        };
        let stream;
        let compressedStream;
        let uncompressedStream;

        console.info('Building CSS');

        if( !install.isSetup() ) {
            console.error('Cannot build files. Run "gulp install" to set-up Semantic');
            return;
        }

        /* unified css stream */
        stream = gulp.src(source.definitions + '/**/' + globs.components + '.less')
            .pipe(plumber(settings.plumber.less))
            .pipe(less(settings.less))
            .pipe(autoprefixer(settings.prefix))
            .pipe(replace(comments.variables.in, comments.variables.out))
            .pipe(replace(comments.license.in, comments.license.out))
            .pipe(replace(comments.large.in, comments.large.out))
            .pipe(replace(comments.small.in, comments.small.out))
            .pipe(replace(comments.tiny.in, comments.tiny.out))
            .pipe(flatten())
        ;

        /* two concurrent streams from same source to concat release */
        uncompressedStream = stream.pipe(clone());
        compressedStream   = stream.pipe(clone());

        /* uncompressed component css */
        uncompressedStream
            .pipe(plumber())
            .pipe(replace(assets.source, assets.uncompressed))
            .pipe(rename(cssExt))
            .pipe(gulpif(compiledConfig.hasPermission, chmod(compiledConfig.permission)))
            .pipe(gulp.dest(output.uncompressed))
            .pipe(print(log.created))
            .on('end', function() {
            runSequence(`package uncompressed css ${org}`, maybeCallback);
            })
        ;

        /* compressed component css */
        compressedStream
            .pipe(plumber())
            .pipe(clone())
            .pipe(replace(assets.source, assets.compressed))
            .pipe(minifyCSS(settings.minify))
            .pipe(rename(minCssExt))
            .pipe(gulpif(compiledConfig.hasPermission, chmod(compiledConfig.permission)))
            .pipe(gulp.dest(output.compressed))
            .pipe(print(log.created))
            .on('end', function() {
            runSequence(`package compressed css ${org}`, maybeCallback);
            })
        ;
        });

        gulp.task(`package uncompressed css ${org}`, function() {
            return gulp.src(`${output.uncompressed}/**/${globs.components}.${org}${globs.ignored}.css`)
            .pipe(plumber())
            .pipe(dedupe())
            .pipe(replace(assets.uncompressed, assets.packaged))
            .pipe(concatCSS(`semantic.${org}.css`, settings.concatCSS))
                .pipe(gulpif(compiledConfig.hasPermission, chmod(compiledConfig.permission)))
                .pipe(header(banner, settings.header))
                .pipe(gulp.dest('dist/'))
                .pipe(print(log.created))
            ;
        });

        gulp.task(`package compressed css ${org}`, function() {
            return gulp.src(`${output.uncompressed}/**/${globs.components}.${org}${globs.ignored}.css`)
            .pipe(plumber())
            .pipe(dedupe())
            .pipe(replace(assets.uncompressed, assets.packaged))
            .pipe(concatCSS(`semantic.${org}.min.css`, settings.concatCSS))
                .pipe(gulpif(compiledConfig.hasPermission, chmod(compiledConfig.permission)))
                .pipe(minifyCSS(settings.concatMinify))
                .pipe(header(banner, settings.header))
                .pipe(gulp.dest(output.packaged))
                .pipe(print(log.created))
            ;
        });
};
{% endhighlight %}

My thoughts behind this attempt were the following: The original task is setup to find all of the relevant  `.less` files based on the configuration files. I wanted to retrieve the `organizations.json` file, iterate through each organization and create a new set of tasks for each one using the configuration files created for each individual theme. I soon found the problem with this approach is that the build process only seems to use the original `theme.config` file. I tried pointing the build to `theme.org1.config`, etc, but it doesn't work and doesn't provide any error.

*I tried a number of iterations on the above attempts with no success. I wasn't willing to give in to the fact that it wasn't possible though without rewriting the whole build process from scratch.*

I finally found my solution. In the end, I ended up modifying the `semantic/tasks/build.js` file to what I've included below. Just before the `module.export`, I'm including the same json file referenced in the last post that contains an array of organizations licensed to use the product. I then created a loop which iterates over each organization and creates three gulp tasks for each.

1. The `theme.config` gets copied to the location the build engine is expecting to find it in.
2. The `build-css` is called.
3. The built css files get copied to a dist location.

The for loop then pushes the names of these newly created gulp tasks into an array. After the for loop finishes, the function ends with `runSequence(...tasks, callback);`. This utilizes the spread operator to place all of the tasks in the tasks array as steps to be run in sequence by gulp. Currently this is the biggest down-side of this approach. As these steps aren't run in parallel, when the number of organizations grows, so does the build time. I tried a few other avenues to have things run in a more parallel approach, but found myself [debugging semantic's](https://stackoverflow.com/questions/44031816/is-there-a-way-to-build-multiple-semantic-ui-themes-in-the-same-project) build system [much more](http://forums.semantic-ui.com/t/how-can-i-build-multiple-themes/462) than I had wanted to, so I stuck with this approach for now. Due to this limitation and the fact that this particular project only uses the styling from the semantic framework, but not any of its javascript plugins, you can see that there are a few tasks I've commented out to speed up the build process a bit.

{% highlight javascript linenos=table %}
/*******************************
          Build Task
*******************************/

var
  /* dependencies */
  gulp         = require('gulp-help')(require('gulp')),
  runSequence  = require('run-sequence'),

  /* config */
  config       = require('./config/user'),
  install      = require('./config/project/install'),

  /* task sequence */
  tasks        = []
;


/* sub-tasks */
if(config.rtl) {
  require('./collections/rtl')(gulp);
}
require('./collections/build')(gulp);

const orgs = require('../../build/licensed-organizations.json').orgs;
module.exports = function(callback) {
  console.info('Building Semantic');

  if( !install.isSetup() ) {
      console.error('Cannot find semantic.json. Run "gulp install" to set-up Semantic');
      return 1;
  }

  tasks.push('build-assets');

  for (var i = 0; i < orgs.length; i++) {
    /* check for right-to-left (RTL) language
       if(config.rtl === true || config.rtl === 'Yes') {
           gulp.start('build-rtl');
           return;
       }

       if(config.rtl == 'both') {
           tasks.push('build-rtl');
       }

       tasks.push('build-javascript');
    */

    /*
        replace tasks.push('build-css');
    */
    const org = orgs[i];

    gulp.task(`copy theme ${org}`, function() {
      return gulp.src(`./src/themes/${org}/theme.config`)
                  .pipe(gulp.dest('./src/'));
    });

    gulp.task(`build css ${org}`, [`build-css`]);

    gulp.task(`copy output ${org}`, [`build css ${org}`], function() {
      return gulp.src(`./temp/**/*.css`)
                .pipe(gulp.dest(`./dist/${org}`));
    });

    tasks.push(`copy theme ${org}`);
    tasks.push(`copy output ${org}`);
  }

  /* runSequence(tasks, callback); */
  runSequence(...tasks, callback);
};
{% endhighlight %}

This all took me a tremendous amount of time(way more than I was planning on) and frustration, but also left me with a great amount of satisfaction. A few key things that really helped me along the way were to remember that with npm, the actual files that are being run are copied to your computer as uncompiled artifacts. This makes going in to them and altering them very easy. While I'm sure there are better ways to debug through gulp tasks utilizing Visual Studio Code, I am unaware of how to do that, so I utilized the tried and true method of `console.log` statements in combination with a number of `JSON.stringify` calls.

Having gotten the build to work in the way I wanted it to, there was still the problem of creating this process in such a way that it could be a reproducable build process. It needed to work for my machine as well as on anyone else's machine(including the build server) that wanted to pull down the solution from source control and work on it. For this piece I used a bit of trickery that I've picked up regarding npm scripts.

I plan on writing a bit more about npm scripts in a later post, but for now, after one pulls down a JavaScript codebase, it is very common practice to have to run the `npm install` command. This tells the node package manager to pull down all the packages that you have defined as requirements for your code to work. You can define a `postinstall` script which npm will run once the installation of all packages is complete. I defined mine as follows `"postinstall": "ncp build/templates/semantic-ui-build.override.js src/styles/semantic/tasks/build.js"`. `ncp` is an npm module used for copying files. All this does is take the modified file shown above(which I placed in a `build/templates` directory) and copies it to the place where I specified for the installation of my semantic files(`src/styles`). Once this is done, when the semantic build task gets executed(`gulp --gulpfile ./src/styles/semantic/gulpfile.js`) our override is in place and the multiple themes will be compiled.

The last caveat I will add is that this was written given the current version of the semantic-ui npm package(`2.2.13` at the time of this writing). It is very possible that semantic-ui will publish a newer version with updates to their build process that could break this override. Currently when you run `npm install semantic-ui --save-dev` it will add that package to your package json with the version `^2.2.13` which means that on your build server, npm will pull down the latest minor release that matches that version. Therefore, npm will pull down `2.2.14` or `2.3.0` automatically on a fresh install if the Semantic UI team has published one of those versions as the latest package version to their package feed, but it wouldn't pull down `3.0.0` if that is the latest package. The most straight forward way to mitigate the issue with the override would be to simply update the version of semantic-ui package in your `package.json` to be `2.2.13`. That tells npm to pull that package version specifically. Then when there are newer versions you can install them(`npm install semantic-ui@2.2.14 --save-dev && npm run postinstall`) specifically and test to make sure the process still works for the latest semantic ui version.
