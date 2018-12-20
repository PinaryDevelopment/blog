---
layout: post
permalink: /visual-studio/templating/multi-project-template
title: "Visual Studio Templating: Robust Multi-Project Template"
excerpt: How to create a robust multi-project template for Visual Studio.
categories: ['Visual Studio', 'template', 'scaffolding']
---

The template that I am aiming to create is supposed to get some input from the user. The intent is to utilize that input in the creation, not only of the project's namespace, but also to help build out the initial base objects. You can find the [full project](https://github.com/PdFramework/Utilities.VisualStudio.ApiProjectTemplateGenerator) on GitHub.

---------------------------------------
*To recap, I would like it to create the base objects and structures that the team would like to encourage in the construction of our micro-service projects. Also, through a minimal amount of work, largely revolving around the validation logic for a given object, the developer should have a deployable solution ready for the Quality Assurance team.*

*Where we left last time was: The wizard launches, gets the user's input and creates a project with all of the files, but the template variable replacement isn't functioning as expected yet.*

---------------------------------------

The first step to get moving forward was an attempt to find a way to get the replacement values to cascade all the way from the user input into the 'child' templates. After spending a lot of time with Bing, I found [a post](http://blogs.msmvps.com/kevinmcneish/2010/09/22/multi-project-visual-studio-template-tricks/) that seemed to discuss exactly what I was interested in doing. Fortunately, there was a commentor that also didn't seem to understand how to utilize the `<WizardData>` element. Even more fortunate was that he ended up [creating a working solution](http://vsix.codeplex.com/) and included that in his comment. This led me to being able to create my first [fully functional multi-template project](https://github.com/PdFramework/Utilities.VisualStudio.ApiProjectTemplateGenerator/tree/ChildWizardCreation). As demonstrated later on, I believe I was able to get closer to the true intent of the blogger, after stumbling across another example project referenced in a StackOverflow answer.

After accomplishing this, I hoped to create a way to update the templates, ensure they still built and then have all of the template files automatically recreated to reflect the changes. I accomplished this through the use of PowerShell. As I'm not terribly familiar with this very powerful scripting language, this took quite a bit of “<a href="javascript(0)" title="Combination of Bing and Google">Bingle-ing</a>”, but I was able to [work it out](https://github.com/PdFramework/Utilities.VisualStudio.ApiProjectTemplateGenerator/tree/TemplateGeneratorCreation). 

A few things to note: I placed this as a Post build event on the Wizards project and made the Wizards project dependent on all of the other projects. This was accomplished in Visual Studio through `Project -> Project Dependencies`. Also due to the fact that it is creating a .zip file, I found a number of different approaches out there, but the most straight-forward one involved requiring PowerShell 5.0 which is available through [Windows Management Framework 5.0](https://www.microsoft.com/en-us/download/details.aspx?id=50395). The net effect is that changes can be made to the 'template' projects and when the changes are saved and built, the true template files will be created and zipped up to a set location that MsBuild will pick up as part of the vsix packaging it does when it builds the 'vsix' Wizards project.

The next step actually took quite a while and a lot of hair pulling until I figured out how to get the vsix to build properly. Thankfully, I found a post on [Stack Overflow](http://stackoverflow.com/questions/26335939/how-to-integrate-a-custom-project-template-and-wizard-into-visual-studio-package/1874522#26406746) that started to point me in the right direction. Utilizing [the project that the answer references](https://github.com/tunnelvisionlabs/VsixWizardSample), I was able to compare and contrast our vsixmanifest and csproj files to determine what was [going wrong](https://github.com/PdFramework/Utilities.VisualStudio.ApiProjectTemplateGenerator/tree/VsixCreation).

I am still unsure how the xml nodes I removed from the csproj got there or why they were causing the pkgdef to not be produced and therefore caused the contents of the vsix to be incorrect as well, but I am very grateful for having a project to compare against. The sample project also brought to light what I think was more the intent of a previous blog post about utilizing a static property in the wizard along with `<WizardData>`. You can see these changes reflected in the `Wizard.cs` implementation as well as the `TemplateGenerator.ps1`.

*A few things to note that helped me along the way. The vsix file is really a zip file. If the vsix doesn't seem to be installing properly, it helped me a lot of unzip the vsix file and see what it actually contained.*

*I worked on creating an internal feed for the new vsix file so that updates could be recognized and installed through Visual Studio's Extensions and Updates tool. I hope to create a post around that in the future as well as it proved more challenging than expected.*
