---
layout: post
permalink: /documentation/standards/project-naming
title: "Standards: Project Naming"
excerpt: What are the internal standards for project naming?
categories: ['standards', 'conventions', 'scaffolding']
---

# PROJECT NAMING

Peineary Development would like to have a standard way of naming our projects to ease development and find-ability across our systems. Systems currently include things like source control, build and deployment systems as well as the databases we utilize.

### HOW TO NAME A PROJECT

There are only a few questions that one needs to ask themselves when naming a project. While this might sound a bit silly, it is usually best to answer these questions with the help of at least one other person. This should preferably be someone not very familiar with the concept that the new project is meant to contain, encouraging the usage of terms that easily make sense to a broader audience.
What business entity is this being developed for?
*Examples: PeinearyDevelopment, ThirdPartyContractingVendor*

What is the broad purpose that this is being developed for?
*Examples: Framework, User Interface, Applications, BusinessComponents*

Is there a more specific, but still broad umbrella concept in which the code under development can fall?
*Examples: Addresses could be on its own, but is a concept that is tied directly to the broader concept of Locations*

Is it possible that this same development concept will be exposed inside a given network as well as outside of the DMZ?
*For example, is there a need for two applications that deal with Addresses, one that will be exposed externally (DMZ) to external customers and one that will be exposed internally on our network to our staff that will provide more sensitive information that will allow them to help in the processing issues?*

What is the specific concept you are developing this project for?
*Examples: Logging, Addresses, Individuals, BlogPosts*

### EXAMPLES UTILIZING THE NAMING PROCESS
A developer is assigned the task of creating a library for standardizing and unifying the way we log events in our applications.

- The main entity this is being developed for is PeinearyDevelopment.
- Since this is meant to standardize the way we approach a certain issue common to all of the coding bases we create, this would apply to the broad umbrella of ‘Framework’.
- There isn’t another broad concept that applies to this.
- This is to be utilized no matter if it is an internal or external application.
- The specific concept being developed is Logging as we hope to utilize it to log exceptions and other events at any desired level of granularity.
Therefore, the full name space would be `PeinearyDevelopment.Framework.Logging`

A developer is assigned the task of creating a library that will encapsulate the concept of Addresses.

- The main entity this is being developed for is PeinearyDevelopment.
- This is meant as a project to encapsulate all business logic relating to a specific business component.
- In a broad sense, this concept falls under the umbrella of Locations.
- This concept represents potentially protected information as we don't want our customers addresses exposed externally, but we would like to build the concept of addresses and maintain all of their associated validation logic through some of our applications to allow our clients to track addresses relevant to their businesses. As such, it is only being exposed internally currently and would need more thought before it would be exposed externally from the organization.
- The specific concept to be encapsulated is Addresses.
Therefore, the full namespace would be `PeinearyDevelopment.BusinessComponents.Locations.Internal.Addresses`

Other possible examples:

* `PeinearyDevelopment.Framework.ApiBases`
* `PeinearyDevelopment.Applications.Internal.Addresses`
* `PeinearyDevelopment.UserInterface.Pager`

### IMPACTED AREAS

**GitHub**

The Project should contain the first two pieces from the namespace.
The Repository name should be the remaining pieces of the namespace.

**TeamCity**

The Project should contain the first two pieces from the namespace.
The Plan name should be the remaining pieces of the namespace.

**Database**

The sql database should be created at the ‘broad umbrella concept’ level. (e.g. Locations)
The sql tables should be named as follows:
 Schema should be the ‘specific concept’ 
 Tables should be the logical objects under that schema.
Example:  [Locations].[dbo].[Addresses]
 (See Sql Naming Standards & Conventions document for more information)

**Infrastructure**

Anything namespaced as ‘Internal’ should only reside on one of the internal web servers. 
If it is a business component, it should reside on the internal web services set of servers.
If it is an application it should reside on the internal web servers set of servers.
Nuget packages should start with the above namespace and then should end with the more specific project name given to that specific piece in the solution.
