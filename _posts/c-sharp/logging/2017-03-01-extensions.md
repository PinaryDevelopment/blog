---
layout: post
permalink: /c-sharp/logging/extensions
title: "Logging: Extensions for ease of use"
excerpt: Describing the design of the extensions for the logging framework.
categories: ['C#', 'logging', 'NLog', 'framework', 'architecture', 'ASP.NET', 'ASP.NET MVC', 'ASP.NET WebApi 2', 'WebActivator']
---

As discussed previously, the objective of this project is to create a uniform centralized way of handling event logging across our applications. Instead of building a logging framework from scratch, the decision was made to create a wrapper around a publicly available library(NLog) that can allow us to tap into the power of the open source community while making minor modifications specific to the needs of Peineary Development. A previous post discussed the core project that was created to wrap NLog.

---
I wanted to take one more post to at least briefly document the rest of the projects that are included in library and their intended use.

---

### Contracts
These really could have been discussed with Core. The contracts project really contains all of the classes that might be used externally from the Logging project to interact with the Logging project. Any of our projects that want to log some information will need to utilize one of the classes provided by the contracts project to do so. Also, utilizing a micro-service architecture, we setup a micro-service that can handle the actual ingestion of and storing of events to a data store. That project expects only to receive an object type defined in this contracts project.

For the initial implementation, we only anticipated/created/exposed three event types. 

* `ActionTakenEvent` - An event that could be logged anytime an action was taken in our system. For instance, an object was created, deleted, updated or an export/import occurred.
* `LogEvent` - This is a more generic type of log event. This could be used to throw any message on the log, be they informational or application errors. This is the type that is used by most of the projects in the Logging solution as the intent is to globally handle errors in a uniform way across our applications.
* `LoginEvent` - An event that is logged upon a user logging into any of our applications. The intent is to be able to track usage.

### NLog.Console
As can be seen with the rest of the solutions, there is very little code as each is meant to encapsulate one purpose. The `NLog.Console` project has the default `Configure` method which, when called, configures logging for a desktop application. It needs to be called at the main entry point of the application and when done, it will make the LogManager available throughout the application as well as setup a handler for any otherwise unhandled exceptions the application might throw through its execution.

### NLog.Web
This is very similar to `NLog.Console` except it doesn't setup a handler for any otherwise unhandled exceptions the application might throw through its execution. That setup is handled in the Http and MVC projects as there are different plugin points for each project type.

### NLog.Web.Http
As can be seen by looking at the code, as we go into the projects that have more lengthy namespace, we are getting to projects that are more specialized and therefor contain much less code. This is to enable users to only be able to pull in what they need and nothing more. `Nlog.Web.Http` contains the few classes that are specific to setting up logging in an ASP.NET Web.Api project. There is a `GlobalErrorLogger` that taps into a hook in the request/response pipeline that enables handling of any exception that is thrown in the application and is otherwise unhandled. The `Configure` method on the `WebApiLoggerConfigurer` then allows this setup to occur as would be done with other configurable objects on `Application_Start` as is commonly done with the route configurations and others.

### NLog.Web.Mvc
`Nlog.Web.Mvc` contains the few classes that are specific to setting up logging in an ASP.NET MVC project. There is a `GlobalErrorLogger` that taps into a hook in the request/response pipeline that enables handling of any exception that is thrown in the application and is otherwise unhandled. The `Configure` method on the `WebMvcLoggerConfigurer` then allows this setup to occur as would be done with other configurable objects on `Application_Start` as is commonly done with the route configurations and others.

### NLog.Web.Http.WebActivator & NLog.Web.Mvc.WebActivator
Both of these projects tap into the power of the [`WebActivatorEx` NuGet package](https://github.com/davidebbo/WebActivator). This package enables the above two hooks to be registered in the request/response pipeline of the application without the need for the developer to actually call the `Configure` methods in the `Application_Start` method. This is really powerful in that it can reduce configuration mistakes and enable ease and consistency of use.

### Sandbox
As a very rudimentary way of creating some 'manual' tests for this solution, I have created three projects in the `Sandbox` directory. These were just ways for me to demonstrate to myself that the Logger would work as expected and provide examples for other developers to know how to pull in and configure the logger in their projects. Unfortunately, they can't be run automatically by a build system and as programmed, require certain setups to be done on the developer's box for them to work, such as having `RabbitMQ` installed.

While this set of libraries has a number of things that are specific to Peineary Development, my hope in publishing this post is that others would be able to find some benefit in seeing how we utilize and wrap the `NLog` library to utilize its code-based configuration, how we were able to extend it through creating a custom `MassTransitTarget` and how we were able to utilize `WebActivatorEx` to automatically and consistently configure these logging capabilities across our projects with minimal setup and configuration effort.
