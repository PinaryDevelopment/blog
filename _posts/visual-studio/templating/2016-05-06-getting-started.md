---
layout: post
permalink: /visual-studio/templating/getting-started
title: "Visual Studio Templating: Getting Started"
excerpt: How to get started creating a project template for use in Visual Studio?
categories: ['Visual Studio', 'template', 'scaffolding']
---

The template that I am aiming to create is supposed to get some input from the user. The intent is to utilize that input in the creation, not only of the project's namespace, but also to help build out the initial base objects.

---------------------------------------
*To recap, I would like to have a project that compiles out of the box. It would be nice if it would also create the base objects and structures that the team would like to encourage in the construction of our micro-service projects. I hope, as an added bonus, to be able to create some of the base unit/integration tests as well so that there are good examples, patterns and practices to follow from day one. Also through a minimal amount of work, largely revolving around the validation logic for a given object, the developer should have a deployable solution ready for the Quality Assurance team.*

---------------------------------------

I created a [GitHub project](https://github.com/PdFramework/Utilities.VisualStudio.ApiProjectTemplateGenerator) for this post and tried to create a branch for each step of the process. I have created links to the branches at the relevant points in the post. I started working off the documentation from Microsoft [here](https://msdn.microsoft.com/en-us/library/ms185301.aspx).

I began by creating a [WinForms app](https://github.com/PdFramework/Utilities.VisualStudio.ApiProjectTemplateGenerator/tree/InputFormProjectCreation) with one form. This was to ease the debugging process while creating the basic interaction between the developer and Visual Studio that was to take place during the creation of a new project utilizing this template.

I then followed the steps outlined to [implement the IWizard interface](https://github.com/PdFramework/Utilities.VisualStudio.ApiProjectTemplateGenerator/tree/WizardsProjectCreation). I added the assemblies as mentioned(there are actually NuGet packages publicly available) and had Visual Studio sign the .dll(this doesn't have to be done if creating a vsix, but for the demo it worked well). Once that was done, I added the .dll to the GAC by running the following from the command line: 
`"C:\Program Files (x86)\Microsoft SDKs\Windows\v10.0A\bin\NETFX 4.6.1 Tools\gacutil" /i "{Full path to assembly}"` The weird thing at this point was that the .dll was located at '*Utilities.VisualStudio.ApiProjectTemplateGenerator\Wizards\\**obj**\Debug*' instead of where I would have expected which would have been '*Utilities.VisualStudio.ApiProjectTemplateGenerator\Wizards\\**bin**\Debug*'.

With all of this done, I was ready to [create the projects](https://github.com/PdFramework/Utilities.VisualStudio.ApiProjectTemplateGenerator/tree/ProjectsToUseAsTemplatesCreation) that I would like to utilize to create my multi-project template. I created all of the projects as a part of the templating solution. As I'll demonstrate later, I wanted to be able to make changes to the projects, ensure they still built and then utilize a PowerShell script to help me actually generate the template files.

At this point, as a proof of concept, I [manually created all of the templates](https://github.com/PdFramework/Utilities.VisualStudio.ApiProjectTemplateGenerator/tree/TemplatesCreation). 

1. In Visual Studio: `File -> Export Template`. 
2. Select Project template and choose the project to create the template from.
3. Click Next, use all of the default values shown.
4. Click Finish.

I then inserted all of the replacement token values I was interested in replacing utilizing the Wizard's replacement dictionary. One thing to be aware of at this point is the naming of your replacement tokens. The common practice in the examples is to utilize strings starting and ending with a `'$'`. This works well, but is also used by other utilities such as NuGet. My suggestion would be to either 'Namespace' your variable names. [I didn't follow this approach for this example as I thought of it after the fact.](https://github.com/PdFramework/Utilities.VisualStudio.ApiProjectTemplateGenerator/issues/10)

I created a .vstemplate manually for a [multi-project template](https://msdn.microsoft.com/en-us/library/ms185308.aspx) and placed that as well as all of the template projects folders in the `'User project templates location'` unzipped. You can find where this is on your machine by going to `Tools -> Options -> Projects and Solutions`.

With the template in the appropriate location and the Wizard .dll placed in the GAC and referenced at the bottom of the template in the `<WizardExtension>` node, you should be able to launch Visual Studio and search for your project name and create a new project utilizing your templates and wizard. As you will note, the wizard launches, gets the user's input and creates a project with all of the files, but the template variable replacement isn't functioning as expected yet.

The next more 'advanced' steps will be outlined in another post to come.
