---
layout: post
permalink: /c-sharp/.net-standard-what
title: .NET Standard - WHAT?!?
excerpt: What is .NET Standard? How does it relate to .NET Core and .NET Framework? Also, why should I care?
categories: ['C#']
---

I watched an entertaining [video](https://channel9.msdn.com/Shows/Careers-Behind-the-Code/Building-Careers-with-Empathy-with-Scott-Hanselman) the other day on Channel 9. This is an interview with Scott Hanselman discussing his career over time. At the end he is asked about his #1 piece of career advice. He states: "Jon Udell told me - 'Don't waste  your keystrokes.'". He continues to discuss this mantra that he has applied to his professional life. If someone asks him a question and the answer is getting a bit long, he says he won't IM or email the answer because "Email is where keystrokes go to die.". Instead he says he will write a blog post and in doing so, "Every additional person that reads my blog post multiplies my power."

As with a lot of what Mr. Hanselman says and writes about, this made a lot of sense to me. I work with a number of different developers in a variety of locations, industries and skill levels. A lot of the work is done via IM(mostly Slack) because it is easy to have a conversation that can span multiple days and enables me to participate in the discussion on my desktop or my phone. A few days after hearing the above advice I got the following question:
> Is it correct to say that .net core implements .net standard?

To which my answer was:
> yes <br />
> more accurately, a given version of .net core implements a given version of .net standard

At first I thought that was it, but my friend continued to investigate and clarify in a chat that went on for ~300 lines over the course of ~2hrs. When we were done I thought...I should make this into a post :). So here it goes...

> Most basically what the heck is .net standard?

There is a ton of digital ink that has been spilled on this topic. I think this goes to show the general confusion that is out there regarding .NET Standard/Framework/Core, etc. [One such post](https://devblogs.microsoft.com/dotnet/introducing-net-standard/) on the .NET Blog simply states: ".NET Standard is a set of APIs that all .NET platforms have to implement. This unifies the .NET platforms and prevents future fragmentation." To be clear, API stands for [Application programming interface](https://en.wikipedia.org/wiki/Application_programming_interface).

In OOP developer speak, .NET Standard is a set of *interfaces*. In fact, if you wanted to look at what is in .NET Standard v2.1 you can do so on [GitHub](https://raw.githubusercontent.com/dotnet/standard/master/docs/versions/netstandard2.1_ref.md). If you do so, you will see a huge file of...well...C#. There are a number of class declarations, but if you peruse through, they are all method/property signatures without any implementations.

> Ok, so what does this mean? Who is implementing these interfaces?

While in reality, anyone could implement the interfaces if they wanted to, in actuality, this is the work being done by a variety of teams at Microsoft. A table of .NET Standard versions and their implementations can be seen on [GitHub](https://github.com/dotnet/standard/blob/master/docs/versions.md) as well. As stated in the document on GitHub, the versions of .NET Standard are *"additive"* and *"immutable"*. Meaning once interfaces have been established, they will not be changed and higher versions of the standard will have at least what was in the previous version.

Now that we have the base information stated, lets proceed to some of the questions that were asked and their answers to better understand what all of this means.

> Does that mean to say that .NET Core is only using libraries specified in .NET Standard?

No. Again, .NET Standard is an interface. .NET Core is a framework used for building applications and a version of .NET Core provides implementations for given version(s) of .NET Standard interface(s). So, if you need something in .NET Standard version X, you will have to use(at a minimum) .NET Core Y to do so, as that is the version of the framework guaranteed to have the implementation for that .NET Standard X interface.

I think this is where things become confusing for a lot of developers, so lets provide some concrete examples to help better illustrate this.

There is a very important point that should be called out at this point to aid further understanding/exploration. **One version of .NET Core can implement multiple versions of .NET Standard**

Before getting specific with Standard/Framework stuff, let's look at a simple C# example that I think will help illustrate this point. Let's say we want to have a few classes that implement logging and we want our application to choose the actual logger class it uses at runtime based on environment. For instance, if we are developing locally, we want to put the message in the Console, and if we are in production, we want to log the message to a database. We could define an interface as such:

***DISCLAIMER:*** *What follows is code that I wouldn't use for real, but is for explanatory purposes relating to the discussion topic. DbLogger contains 'inline' sql which one shouldn't do for security reasons. Actual instantiation of classes would most likely be handled by a DI container in the real world, etc, etc.*

{% highlight csharp linenos=table %}
public interface ILogger
{
    void Log(string message);
}
{% endhighlight %}

And classes as follows:

{% highlight csharp linenos=table %}
public class ConsoleLogger : ILogger
{
    public void Log(string message)
    {
        Console.WriteLine(message);
    }
}

public class DbLogger : ILogger
{
    public void Log(string message)
    {
        using (var connection = new SqlConnection("CONNECTION_STRING"))
        {
            connection.Open();

            using (SqlCommand cmd = new SqlCommand($"INSERT INTO Log(Message NVARCHAR(MAX)) VALUES ({message})"))
            cmd.ExecuteNonQuery();
        }
    }
}
{% endhighlight %}

Then in our application we could do something like:

{% highlight csharp linenos=table %}
ILogger logger = Environment.GetEnvironmentVariable("Environment") == "local" ? new ConsoleLogger() : new DbLogger();
logger.Log("Log me a test message.");
{% endhighlight %}

Now let me ask a question...Are `ConsoleLogger` or `DbLogger` limited to only one method or could they include methods other than the one in the interface `ILogger` as provided above?

The answer is that they are not limited to only that one method. An interface is a contract of sorts. If a class implements an interface, then it is 'contractually obligated' to provide what is stated in that interface. Both classes above do this as they both provide implementations of a method called `Log` that takes one string as a parameter and returns nothing. BUT, nothing says that `ConsoleLogger` can't also have a `LogError` method as well. The following code is still valid.

{% highlight csharp linenos=table %}
public class ConsoleLogger : ILogger
{
    public void Log(string message)
    {
        Console.WriteLine(message);
    }

    public void LogError(string message)
    {
        Console.ForegroundColor = ConsoleColor.Red;
        Console.WriteLine(message);
        Console.ResetColor();
    }
}
{% endhighlight %}

Take this understanding and now apply it to .NET Standard/Core. .NET Core 1.0 implements .NET Standard 1.0 - 1.6. What this means is that the implementations provided in .NET Core 1.0 contained more than what was defined in the .NET Standard 1.0 interfaces. As .NET Standard was growing, codified and versioned, it added more interfaces which .NET Core 1.0 already had implemented. For instance, `ConcurrentBag`(which is a thread safe collection) was added to .NET Standard 1.1, but already was implemented in .NET Core 1.0. Alternatively, .NET Framework 4.5 implemented .NET Standard 1.0 and 1.1, but not 1.2. To utilize all of the interfaces in .NET Standard 1.2, the target .NET Framework for an application would need to be 4.5.1.

> Hopefully now you are at least thinking: Ok, conceptually this is starting to make sense, but when/why do I need to care? I don't plan on thinking through all of the method calls I plan on making in my application and then hunting through .NET Standard versions to make sure they all exist there before I start development.

> Also, how does Visual Studio have an option to create a class library using .NET Standard if all it is is a set of interfaces?

These questions really go hand-in-hand. Let's start with the second one and work our way back to the first one. In the world of .NET, our code gets compiled to a `.dll` or a set of `.dll`s. There needs to actually be some sort of executable in order for the code in the `.dll`s to get executed on a machine though. Visual Studio provides an option to create a class library 'using .NET Standard' because the output of that class library on build will be a `.dll`, not an executable.

> OK...sounds like semantics?

It's really not though. In order for your code to build and create a `.dll`, it needs to have the definitions for the interfaces it is relying on but doesn't need to actually have the implementations of those interfaces. Back to our C# example above, if we wanted to get this line to compile:

{% highlight csharp linenos=table %}
ILogger logger = Environment.GetEnvironmentVariable("Environment") == "local" ? new ConsoleLogger() : new DbLogger();
logger.Log("Log me a test message.");
{% endhighlight %}

We could refactor it a bit to this:

{% highlight csharp linenos=table %}
public interface ILogger
{
    void Log(string message);
}

ILogger logger = null;
logger.Log("Log me a test message.");
{% endhighlight %}

Now of course, this will throw an `Exception` at runtime, but the code will compile.

Back to our library, by selecting .NET Standard for it in Visual Studio, we are declaring that our library will only depend on methods declared in the version of .NET Standard we have selected for our library. So, if we select .NET Standard 1.0, our library code won't be able to use `ConcurrentBag` because that wasn't added to .NET Standard until version 1.1. This is so EVEN IN A CASE where the application executing this library is a .NET Core 1.0 application(which implements .NET Standard 1.0-1.6, meaning that it actually has an implementation for `ConcurrentBag`).

When/why do I need to care? The most common scenario I've found for when developers really need to be aware of all of this is if you are a library maintainer(think AutoMapper, Serilog, etc) or you are in an organization that is trying to migrate its applications from a version of .NET Framework to .NET Core.

The '.NET' recommendation for deciding which .NET Standard version to use for a library is: "Pick the lowest one that has all of the interfaces required in your library." This is great and makes a ton of sense. I have found that it can be tedious to determine what version this might be though for any given dependency.

If you are migrating an application, I have found that it is important to be mindful of what .NET Framework version your application is currently on, find the highest .NET Standard implementation that that version of the Framework adheres to and create libraries of 'migrate-able' code based on that.

> Ok, so rewind a bit. You're saying the only reason why my library can run `Console.Writeline`(for example) is because the application running the library is using a concrete framework that actuality implements `Console.Writeline`?

> yes

> What is the point of having the other 2 options for a class library (.NET Core and .NET Framework)? Shouldn't .NET Standard suffice for anything?

Not exactly. Again, the frameworks have to implement the interfaces of .NET Standard AT A MINIMUM. But they aren't limited to those interfaces. They can(and do) give you more above and beyond what is declared in the standard. In fact, this is something I've found you need to keep in mind while migrating applications from .NET Framework to .NET Core. .NET Core is built to be cross-platform. This means that it can be compiled to run on Linux, Mac or Windows(others as well). If your current application is logging events to the Windows Event Log, there won't be a direct migration path as the 'Windows Event Log' doesn't make sense on Linux or Mac. These types of dependencies will need to be isolated in .NET Framework libraries to enable the current application to function as usual while a .NET Standard version is worked on that could execute similarly in the new & old applications.

It is understandable why this topic can seem a little obtuse. I hope that this article was able to help clarify some of these topics!
