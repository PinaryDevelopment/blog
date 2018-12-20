---
layout: post
permalink: /c-sharp/logging/the-problem
title: "Logging: The Problem"
excerpt: Describing why and how I approached building a logging framework for an organization.
categories: ['C#', 'logging', 'NLog', 'framework', 'architecture']
---

One of the companies I've worked for is a fairly small development shop. One of their applications was a fat-client, WinForms application that has had little to no overarching project architectural work done to it. In helping rethink the architecture of their solution(s) and related data structures, one thing the team noticed immediately was the haphazard and non-uniform way the application approached error handling and message logging.

Uniform and proper error handling and message logging can go a long way, when problems start to occur, to quickly identify and hone in on the problematic areas of code as well as the related data causing the issue. Without having a uniform approach, we quickly found ourselves spending a lot of time trying to locate and reproduce bugs that were causing our users frustration.

One of the more common methods utilized in the application to handle this message logging was to send an email to a party in the organization affiliated with that area of the application(of course the email addresses were stored in the config files). As you can imagine, this was difficult to maintain as people moved around and most of the emails began to be ignored as the volume of them increased over time. There were a number of issues where as we started to dig into them we would hear, "Oh, yeah, we get an email every time that occurs, but we just ignore it. Things seemed to keep functioning correctly." Of course, there are a number of ways to mitigate some of the above issues(create a distribution list email to send to instead of an individual, store those values in a database, routinely review the inboxes to find recurring errors), and if all of the errors were handled this way, I might have advocated to mitigate before creating a different approach. Another consideration though was if we wanted to get trace logging or log informational messages, emailing all of them didn't seem to be a great approach. A nicety to have as well would be the ability to dynamically change the level of logging output from any given application as the need arises.

I started to dig-in to creating a uniform approach to logging as well as a guide on exception handling for our applications. I wanted the solution to be something we could utilize across desktop and web applications and I wanted to provide a standard setup that could be dropped into any project and allow the developer to start utilizing the library with little or no additional setup. I also wanted to be able to extend the framework as the need arose and allow for overriding of the default configuration provided.

I started looking into the options available and decided that I didn't want to write my own from scratch. There were a number of reasons for this.
1. Why reinvent the wheel?
2. While more generic libraries tend to be larger and more clunky than custom solutions, they are also more battle-tested and hardened in a lot of ways.
3. Generally you can find a solution that has a lot of flexibility. Due to the broader audience, it needs to provide this flexibility to remain relevant.
4. This was probably one of the biggest factors in my decision: DOCUMENTATION!!!

Yes, you read it right, documentation. You can read [my thoughts on the documentation](/software-documentation-the-beauty-and-the-beast). In a nutshell, I think its important, but also realize the difficulty in providing quality, up-to-date documentation for the software we right. Being provided with a majority of the documentation on the core library is a big plus in my book. That allows me to spend more time standardizing and documenting the good patterns and practices I would like to see utilized across the organization regarding how we utilize these libraries.

After looking around for different options for libraries in this space for the .NET landscape, I chose to go with [NLog](http://nlog-project.org/). It seemed like a very robust option with decent documentation that would allow me to realize the above stated goals. In subsequent posts, I hope to detail some of the decisions I made, document the wrappers created and how we plan on utilizing them across our organization.

The code for these posts will be located on a [GitHub repo](https://github.com/PdFramework/Logging).
