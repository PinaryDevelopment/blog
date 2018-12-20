---
layout: post
permalink: /architecture/patterns/service-oriented/anatomy-of-a-micro-service
title: Anatomy of a Microservice
excerpt: Documentation of the project organization and division of responsibilities of a micro-service.
categories: ['architecture', 'patterns', 'micro-services', 'SOA', 'Service Oriented Architecture']
---

**BUSINESS COMPONENT OVERVIEW**

Service Oriented Architecture (SOA) is an architecture that is built upon many smaller composable modules generally referred to as micro-services. At PeinearyDevelopment, we refer to these services as business components, as the intent is to build them up around components(mostly components that are known and, that when put together, make up a business). This document is intended as an overview to the various projects contained in a business component and their intended purpose.

**BROAD OUTLINE**

As discussed in the [Service Oriented Architecture Overview](/architecture/patterns/service-oriented/overview) document, from a very generic perspective, the main breakdown of the architecture we would like to utilize to create applications would be in line with the following diagram.

![VENN Diagram of the layers in SOA](/assets/images/SOA_VENN.png "VENN Diagram of the layers in SOA")

Each business component resides in the BLL layer and contains a piece of the DAL and the collection of all of the business components together is what defines the BLL.
Each business component itself could be represented by the following diagram:

![VENN Diagram of the layers in micro-service solution](/assets/images/BC_DIAGRAM.png "VENN Diagram of the layers in micro-service solution")

The functionality of the business component exists in the four main projects: Contracts, Business Logic Layer, Data Access Contracts and Data Access Layer. These are supported by the Api Clients, Unit Tests and Integration Tests projects as will be detailed below.

For the purpose of this document, we will discuss a fictitious Locations business component solution, to allow us to provide some examples of what each of these projects in a solution might encapsulate. As a business often deals with many different entities that all might contain location information (business address, home address, etc.), we would like to have one place where all of our knowledge of and business rules regarding a location could be encapsulated and uniformly enforced. To this end, we have decided to create a locations business component.

**CONTRACTS**

Simply put, this project contains the objects and interfaces that are exposed externally by this business component.

What does this mean? Utilizing our locations business component example: A location could consist of a type (business, home, etc.), address line 1, address line 2, state, country and zip code. There are probably a few more metadata points we would like to track with that model as well such as an id, valid from and thru dates, created by and last updated by. That being the case, the contracts project could contain files describing the Location, Country and State classes as well as one for the AddressType enum definition.

Also, we would like to provide ways for various applications to interact with these objects. Some applications might only want to retrieve location information relating to their entities of interest while other applications most likely will need to be in place to manage the locations in our system. The contracts don’t contain any implementation details, only the definitions of these interactions.  That being the case, we could create one or more interfaces in the contracts that discuss the get, create, update, delete and search actions that we would like to expose for applications' interactions with these components.

The purpose of the contracts is to create a loose coupling between this business component and anything desiring to interact with a component of this type. What this means is that the actual implementation of the BLL could be an http endpoint utilizing a Asp.Net WebApi or a SOAP endpoint utilizing a WCF service or any other type of underlying implementation and as long as that implementation adheres to the stated contracts provided by this project, no application that interacts with this component will have to know or care.

**BUSINESS LOGIC LAYER (BLL)**

This project in the solution should contain the implementation of the interactive actions that can be performed on the objects encapsulated in this business component. All of the methods defined in the contracts' interfaces should be implemented here. In addition to this, the business logic layer should enforce the business rules around these objects. This is generally referred to as validation. The project should enforce that the data conforms to a standard format and are within an acceptable range of values. In our implementation, this project is an ASP.NET WEB.API2. Acting as the BLL, it also contains all of the validation logic for a resource(s) located at its endpoint.

In our locations example, there are certain concerns that we would like to address around the location objects. For instance, to create a state or a country, we would probably want to enforce it has been given a name. For retrieval of a location, we might want to validate that the user performing the action has permission to view that location and if not, return an ‘agnostic’ error message so that we aren’t inadvertently providing information to the user about the existence of a location that they don't have access to.

**DATA ACCESS CONTRACTS**

Similar to the contracts, as talked about above, this project contains the objects and interfaces that are exposed externally by the data access layer of this component.

What does this mean? Utilizing our location business component example: The location we discussed above has a relationship with a state and a country object. We would most likely, in a relational data store model, want to create one table that stores locations, one for states and one for countries. We would also like to create foreign key relationships between them to further enforce this relationship. While for the most part, the objects in the data access contracts will mirror those in the contracts project itself, they could also contain additional information relating solely to the data model representation of these objects. For instance, utilizing Entity Framework, these objects would most likely contain attributes on them that could represent constraints or additional properties that EF utilizes to create table information such as virtual properties that represent an object related through a foreign key.(Technically speaking, these attributes are in the `System.ComponentModel.DataAnnotations.Schema` namespace, so this project wouldn't actually be dependent on EntityFramework.)

The purpose with these contracts also is to create a loose coupling between the business logic implementation and that of the data access implementation.  What this means is that the actual implementation of the data access layer could be a relational data store, a text file or a document data store or any other data persistence mechanism and as long as that implementation adheres to the stated contracts provided by this project, the business logic layer won’t require being updated to account for those changes. 

**DATA ACCESS LAYER (DAL)**

This project contains the actual implementation of the system's data access. All of the knowledge of and interaction details required to work with the data store of choice for this business component should be contained in this project. For instance, if the project is utilizing Entity Framework to interact with a sql data store, the DbContext and Migration objects should be in this project. The opening and closing of connections performed through those context objects and their related queries should be contained within here as well. If ADO.NET is being used for a similar purpose, all of the connection, command and reader management should be contained here.

**UNIT TESTS**

The unit tests project serves two main purposes.

1.	It encourages crafting code in such a way that it is cleaner, more readable and easier to maintain code. 
2.	It provides a safety net for future revisions and enhancements.

How does this project encourage or promote these purposes? This has been written about much more extensively and eloquently by others. I don’t intend to repeat those thoughts here, but would like to include good reference points for further reading.

* [testdouble.js](https://github.com/testdouble/contributing-tests/wiki/Tests%27-Influence-on-Design)
* [Erik Dietrich](http://www.daedtech.com/tag/tdd/)
* [MSDN how-to walkthrough](https://msdn.microsoft.com/en-us/library/dd264975.aspx)

Through breaking the test suite into two projects, we intend to utilize the unit tests project as a place to put all of our tests that focus on small units of localized code. All external dependencies that are utilized by that local block of code should be mocked. For instance, any calls to the DAL or to other APIs through their ApiClients, should be mocked out via their interfaces. 

Each test in this project should be small and focus on only making one assertion. For instance, one test could assert that calling the BLL create method on the country api with a country object that has no name will produce an HTTP response with a status code of 400. Another test could assert that the same scenario returns a meaningful message with the HTTP response to let the caller know why the POST failed.

**INTEGRATION TESTS**

Since the unit tests are asserting the correctness of the small units of code, that when composed together make a complete business component, the integration tests are meant to assert the correctness of the full integration of those smaller units of code. For this project, no dependencies get mocked out. These tests make actual calls to the exposed REST endpoints and those calls are executed all the way to the database.

This creates another layer of automated testing for our business components as well as another layer of affirmation that the code executes according to the desired specifications.

The intent is that the unit tests should be run by the developer prior to check-in of any code and the automated build system should run all tests in the unit tests project as well prior to build artifacts being packaged for a deployment. The integration tests on the other hand should be executed immediately following a deployment of a business component, to assert that the deployment succeeded and that the micro-service is functioning as expected. These tests should be responsible for setting up all necessary data for the test as well as cleaning up all said data after the tests have been executed.

**API CLIENTS**

As mentioned before, the above projects really contain all the necessary parts for making the business logic layer component serve its purpose. The api clients project is really a ‘nicety’ that we would like to provide so that any application that wants to utilize the component in C# has a way to do so through a strongly typed means. In general terms, an HTTP endpoint provides access to ‘resources’ through a set of ‘actions’. This is all done through ‘magic strings’, meaning, to interact with these endpoints, one would need to know the exact URI to invoke to perform an action on a resource and the exact shape of a resource in order for that action to be performed as expected. This is nice in that there are many ways that these resources can be accessed and actions can be performed. It is really a language agnostic way of exposing these business components.

While the HTTP concepts don’t map exactly to our contracts, the api client provides a means to meet these same goals in a strongly-typed manner. In our parlance, our resources roughly equate to the C# objects detailed in the Contracts project and the actions are lightly wrapped by the methods declared through the Contracts’ interfaces.
