---
layout: post
permalink: /building-a-framework
title: Building a Framework
excerpt: Thoughts on how to approach creating an organization's software system's architecture.
categories: ['architecture']
---

I've been spearheading quite a few projects at the moment focused on a software system's architecture. In doing this I've aimed to create a number of libraries that will establish a solid foundation and standard implementation to enable the building of the system's components going forward.

After defining a broad architectural model that we would like to utilize while building new applications and to start migrating the older application, the first step our team took toward this new vision was to perform a gap analysis of sorts. We wanted to take a look at the architecture of the system in place. What common/core libraries exist? What common/core functionality exists in multiple locations throughout the code base with multiple implementations. 

Sadly we found that there was very little code reuse(other than a number of bad patterns and practices being copied and pasted all over the place) and found multiple implementations for most of the 'core' pieces of 'plumbing' we would expect to find in more robust applications. As you can expect, with all of this, we found many different fundamental misunderstandings/misuses of exception handling and other base level language constructs.

Seeing this, we wanted to start building a set of core libraries that we could utilize to encapsulate some of these common constructs to allow for code reuse and a more consistent code base.

On a fairly high level, we identified the following building blocks that we would like to put in place to help the organization move forward(these are in no particular order):
1. Standards documentation
2. Code reviews
3. A set of base classes
4. Project Templates
5. Messaging Queue
6. Logging Framework
7. Private Repositories
8. Authentication/Authorization Framework
9. Better system health monitoring
10. Automated deployment system

There were a few others that were specific to our environment that I didn't enumerate above. A number of the above building blocks were pieces of code that we decided would best be encapsulated in a shared set of libraries.

I believe that a good first step for most of these libraries is to utilize what is already out there. It would seem reasonable to ask, "Why reinvent the wheel?". In a number of instances that I've seen as well, the main motivation for a developer to want to 'roll their own' is a bit of arrogance, "I/we know better?" My personal opinion is that, when you can utilize an active open-source library, in most cases you should. There are many more people utilizing that library, experiencing it's 'hidden features' and working to fix them. 

That being said, I am aware that there is a counter argument to this logic. Since it is trying to work for everyone, it can be slower and clunkier than one that would be written focused on your organization's specific needs. To me, this is premature optimization. In most cases, the open source option should work well for you. Spend your time delivering software that differentiates your company in the marketplace and only write your own 'optimized' implementation if/when you have to.

My approach to these libraries has been as follows:

1. Research a few choices first. Read through some of their documentation. Look into the repository a bit to see the number of issues/commits and most recent date of commits.
2. Choose one that seems to be a good fit given the problem space. To evaluate this, I look at current capabilities and extensiblity points.
3. Create a test project utilizing the library to demonstrate its abilities and to 'kick its tires'.
4. At this point, I usually present my findings to a group of my coworkers. 
  * This allows us to discuss the library by looking at a concrete example of its use.
  * We can also throw out ideas for different ways of utilizing the library or other features we would like in the library for our continued use of it.
5. Assuming that we are in agreement that the library will meet our needs, I will proceed to create a wrapper library that extracts all of the useful code from the test project and provides a simple/consistent interface to program against.
6. Create a NuGet package for the library which allows for easy importation into any new component projects.
7. Document. 
  * This will include references to key places in the wrapper library as well as the underlying open-source codebase. Places to start looking if and when we decide we would want to extend or change it in the future. 
  * Documentation should also include the basic core pieces that most people will want to know about this wrapper library when utilizing it in one of their projects.

I firmly believe in the mindset of 'Why reinvent the wheel?' and am happy to utilize projects that are not mine. In creating these libraries, I've encountered quite a bit of success but have had a number of struggles and stumbles along the way. I hope to utilize a number of posts outlining these projects. I'm not attacking them in any particular order. As I find time though I would like to document the process I took and some of the struggles along the way to help me remember what I've tried before as well as to hopefully help someone else that finds themselves along a similar journey.
