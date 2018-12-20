---
layout: post
permalink: /fullstack/multi-themed-application/build-scripts
title: "Multi-themed Web Project: Build Scripts"
excerpt: Describing the approach used to create build scripts to aid in the on-boarding of new organizations to the web application with multiple themes.
categories: ['Typescript', 'Angular', 'Semantic UI', 'Node.js', 'ts-node', 'architecture']
---

As can be seen from the previous posts, currently, there are a number of files required to get a build working for a new licensed organization. They each have conventions that need to be followed in terms of naming, placement in the project or contents. This can become a bit of a nightmare when bringing a new organization on board. As a developer, I feel as though it is important to utilize our talents not just for our external customers, but to remember the internal customers as well(and yes, that does even mean we the developers). The next piece I wanted to touch on then, was the creation of some 'tooling' or other scripts that could be used to automate most of the setup. Outside of easing the setup process, it also creates a documented(even if only in code) and reliably repeatable process with which to setup a new organization, saving a lot of headaches down the line.

With the current design and setup, the bare minimum we need to setup a new organization are the organization's name that will be used at runtime to determine the directory to get the css files and two colors that can represent their brand's primary and secondary colors. The `build/setupTheme.ts` script was created just for that. The script can be run with the following command: `node_modules/.bin/ts-node build/setupTheme <THEME_NAME> <PRIMARY_COLOR_NAME>:<PRIMARY_COLOR_VALUE> <SECONDARY_COLOR_NAME>:<SECONDARY_COLOR_VALUE>`. The script grabs the arguments passed, does some minor validation on them, does some string replacement on the files in the `/build/templates` directory and then places them in the correct location in the project. Once those files are in place, the next ui build will generate the appropriate css files and place them along with the rest of the generated files in the organization's `dist` directory. The code can be seen [here](https://github.com/PdUi/semantic-ui-build-multiple-themes/blob/master/build/setupTheme.ts).With this in place, setting up a new organization on the platform can be done in a matter of moments.

Another common task that should be addressed during any automated build of a front-end project is referred to as 'cache-busting'. Usually this is accomplished through adding a hash or part of a hash value of the file's contents to the file name. As browser's have developed, they do certain things to help us and our websites be faster and more efficient. One of the methods they employ is caching of 'static' files. File types like css, js and images usually fall into this rubric. That way, when the browser loads the html, if it sees a file it has loaded in the past, it can just pull it out of cache instead of reloading it from the server. This is GREAT for us and our sites...if the content in those files hasn't changed. What if we have added new functionality, or fixed a bug though? If the file name remains the same as it was before that update, odds are, your browser will serve up the version of the file it has cached locally instead of getting the updated version from the server. If we add a hash value that has been calculated from the contents of the file itself to the filename, then the browser can still serve up the cached file assuming the contents of the file haven't changed because it will have the same hash value. If the contents have changed though, the updated html page will link to the new filename and the browser not finding that in cache, will request it from the server and the user gets the best of both worlds!

As has been described in a previous post, the project has been setup to utilize the power of Semantic UI's styles framework for the majority of the styles applied and which is augmented by a few project specific `.scss` files. Since we had to modify the semantic build to provide us with multiple different themed builds, that process had to include adding a hash value to the file names. The custom `.scss` styles that we include for the project needs this to be a part of their build as well. As such, both processes(`build/buildScss.ts` and `build/cacheBustAndCopySemanticFiles.ts`) were updated to include the `cacheBuster.ts` function. The contents of which are as follows:

{% highlight typescript linenos=table %}
import { rename } from 'fs';
import { fromFile, HashaOptions } from 'hasha';
import { basename, dirname, join } from 'path';
import { handleError } from './handleError';

export const addHash: (filePath: string) => void = (filePath: string): void => {
    const minCssExtension: string = '.min.css';
    const fileName: string = basename(filePath, minCssExtension);
    const hashaOptions: HashaOptions<'base64'> = { algorithm: 'md5' };

    /* tslint:disable:no-floating-promises */
    fromFile(filePath, hashaOptions)
        .then((hash: string | null) => {
            const hashFileNameLength: number = 20;
            const hashedFileName: string = `${fileName}.${hash.substr(0, hashFileNameLength)}${minCssExtension}`;
            const directoryPath: string = dirname(filePath);

            rename(
                filePath,
                join(directoryPath, hashedFileName),
                handleError
            );
        });
};
{% endhighlight %}

It uses some standard node libraries to perform some common path and file access functions and then leverages the library `hasha` to calculate the hash for the file contents.

After these are run, the `copyFiles.ts` function gets executed which takes the front-end files that have been created and places them in the directory that ASP.NET Core utilizes by default(`wwwroot`) to serve static files from.

With all of this in place, a `clean` function was necessary as well. While it is a good practice to have something in place to clean out the directory from all compiled files before a new build, in order to ensure that only the files you most recently built are present, in this instance it was quite necessary. Again, as discussed in a previous post, the website was designed to look for/load the files in a directory that is specific to each institution. Since these files are hashed, it uses pattern matching to load those files instead of being programmed with exact file names. This works well, but if you have artifacts from multiple builds in the same directory, then the results loaded into the browser could be unpredictable as you won't be assured that the latest version of that file is being loaded.

This ends the series focused on how to have a Semantic UI powered front-end with multiple builds in one web application supporting a set of licensed organizations.
