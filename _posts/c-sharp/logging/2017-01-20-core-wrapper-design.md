---
layout: post
permalink: /c-sharp/logging/core-wrapper-design
title: "Logging: Designing the Core Wrapper"
excerpt: Describing the design of the core wrapper for the logging framework.
categories: ['C#', 'logging', 'NLog', 'framework', 'architecture']
---

----
*Brief Recap*
The objective of this project is to create a uniform centralized way of handling event logging across our applications. Instead of building a logging framework from scratch, the decision was made to create a wrapper around a publicly available library([NLog](http://nlog-project.org/)) that can allow us to tap into the power of the open source community while making minor modifications specific to the needs of Peineary Development.

----

# NLog
The NLog project provides a very robust way to manage loggers as well as their ways and means of logging. It provides an internal logger(I know, meta!) that handles logging events specific to the logger. For instance, it logs when any given logger is started up.

The NLog project provides the means of configuring the logger through an XML file or through code. I tend to prefer code over configuration files, so I've created this project to provide a default setup for the configuration of NLog across our applications.

### Target
>Targets are used to display, store or pass log messages to another destination. NLog can dynamically write to one or multiple targets for each log message.

A target is an object that contains information relevant to the way and means a given event should be logged. The default target for NLog is a `NullTarget` which does nothing. It completely throws the event away. They also provide a `ConsoleTarget`, which prints event information to the Console and a `FileTarget` which writes event information to a file on disk. A full listing of their targets can be found on their [wiki](https://github.com/nlog/NLog/wiki/Targets).

There is also a great story surrounding extending NLog through creating [custom targets](https://github.com/NLog/NLog/wiki/How%20to%20write%20a%20custom%20target).

# PeinearyDevelopment.Framework.Logging.NLog
### Core
This project provides the essential pieces for setting up logging in one of our applications. 

### Logger Configurer
The LoggerConfigurer’s Configure method is the main entry point for setting up a logger for one of our applications. This uses a `FileTarget` to log its events.

*The default location for any file log created through a `FileTarget` is located in `C:/logs`, but can be configured through setting the value for the AppSetting with the key `Logging.LogsPath`.*

The InternalLogger logs when any logger starts up as well as when a logger target fails along with the exception and what target it falls back to following that failure.

This method also creates a target manager, the default targets as well as the rules that applies to each as described below.

### Target Manager
The target manager is responsible for creating a 'wrapper target' of type `FallbackTarget` as well as properly creating and prioritizing the targets contained within the fallback target for every event that the logger is responsible for logging as well as handling the lifetime of those loggers. The `GenerateDefaultTarget` method takes an array of target types to attempt in the order they should be attempted.

### Logging Rules
A logging rule, defines which logger should log what and when. A log rule defines at what level and above(Log levels listed in descending order: `Fatal`, `Error`, `Warn`, `Info`, `Debug`, `Trace`) a given target should log an event and through which logger the event should be processed.

*The default log level is set to `Warn`, but is configurable through setting the value for the AppSetting with the key `Logging.LogLevel`.*

Any event sent to the LogManager will go through all of the rules to determine which rule(s) apply to it. There is a property on the rule called `Final` which will break that chain. If `Final` is set to `true`, then as soon as an event hits that rule and is applicable to that rule, that rule will process the event and no other rules will be evaluated for their applicability to that event.

### Custom Target
A custom target that we developed is the `MassTransitTarget`. This target utilizes the MassTransit project to publish events it is given to a messaging queue. This will allow the application to publish events to the queue and forget about them.

*The url for the message queue to publish to needs to be configured through setting the value for the AppSetting with the key `Logging.Endpoint`. The username and password for connecting to RabbitMQ(the default messaging queue used behind our systems) will default to username: `guest` and password: `guest` which are RabbitMQ defaults for ease of development. In other environments, those values should be overridden through the values for the AppSettings with keys `Logging.Username` and `Logging.Password` respectively.*

There will then be a project containing a listener for these events which will contain the logic relating to if, how, when and where to persist the events received.
The default target generated for our loggers is of the type `FallbackGroupTarget` which takes an array of targets and utilizes them in such a way that it will try the first one first and only try the next one in line if the first one fails. Our default fallback order is: `MassTransitTarget` and then `FileTarget`.

### Log Manager Extensions
To ease proper use of this library, we also created a few log manager extensions that properly generate a `LogEventInfo` object and handle which Logger a given type is logged to as well as at what level.

While this article has described how we designed our core pieces of the logging wrapper, a subsequent post will be written to discuss further projects we created to ease their use even more across our projects, regardless of whether they are Console, WebApi or MVC web apps.

The code for this post can be found on [GitHub](https://github.com/PdFramework/Logging/tree/master/NLog.Core).
