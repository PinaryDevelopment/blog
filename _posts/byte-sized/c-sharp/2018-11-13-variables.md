---
layout: post
permalink: /byte-sized/c-sharp/variables-I
title: 'Byte-sized C# Programming: Variables'
excerpt: 'C# topics: What is a variable? How do I program with variables? What does it mean when I declare and assign values to variables?'
categories: ['byte-sized', 'c#', 'variables']
---

<aside>Getting started with programming can prove to be very overwhelming as there is so much to learn. The byte-sized series is intended to break topics into small, digestible, easy-to-understand chunks.</aside>

> To set the scene...I travel occasionally for work. There are those in our company that travel more than me and those that travel less. Let's assume the CEO decided he wanted to create his own rewards program for the company. He wants to create a program that can run on each employee's computer(for now) and will track their miles traveled throughout the year.

Let's dive right in. One of the first things I'll need in my program is a way to store an employee's mileage traveled. For this, we will utilize a variable.

> What is a variable?

At a high level, variables can be likened to containers. A container is an object that is meant to hold something else. They come in all different shapes and sizes. For example, you might get a container from a restaurant to take your food home in. You could also have a container in your basement that has winter clothes stored in it. You wouldn't use the clothes container for the take out food and vice versa. You might eat your take out food though and then reuse the container to hold the left-overs from last night's dinner. You also might donate your winter clothes to a charity and decide to store all of your sport's supplies in it instead. It is just a container to hold stuff in.

When a computer runs a program, the program needs to tell the computer that it needs to store a value and how much space it needs to store that value in. Glossing over a TON of details, computers have memory which is just a series of zeros and ones(known as bits). When a program runs, and it needs to remember a value for later use, it requests a 'container' of memory from the computer to save that value in. That container is filled with a series of bits that represent the value stored there. 'Declaring' a variable is the program's way of requesting a container to store a value in.

> What is 'declaring' a variable and how do you do it?

One might attempt to do this by writing: `"Computer, give me space for a mileage variable."`. When this doesn't work, maybe we would think we need to be more polite and would add `"please"` at the end. After all computers have to be polite and would expect the same from it's programmers, right? Actually, we can be as nice and polite as we want, but the computer has no idea what to do with the above. In reality, we don't communicate directly with the computer at all. When we write code, we write something that a person could read, but not a machine. What we write, another program reads and that in turn creates code that we could read only with great difficulty, but is easy for the computer to understand.

For our example, the mileage variable declaration could look like this:

<pre>
<code>
int employeesMileage = 0;
</code>
</pre>

> I thought you said that "when we write code, we write something that a person could read". I recognize the words: employee's & mileage *AND* I see the number `0`, but it still looks like gibberish, what gives?

It is true that we write code in a way that a person could read it, but since there is another program that needs to read it and translate it for the machine's readability as well, there are certain rules the writing has to adhere to for the translation program to work properly. My native, first-language is English. I've met and had many interactions with those whose first-language isn't English. Often times, there are certain phrases or sentences that they say that I might be able to understand, but definitely aren't proper English. What we write has to be proper c#, or the translating program(called a compiler) won't be able to create something that the computer can read.

Now, let's break down the above statement to better understand what it means. To allow us to write code that 'a person can understand', we are allowed to give certain things in our program names. Variables are one thing that we are able to name. We can't however give variables any name we want. This is one of those things that has to be proper c#. There are a few rules, but for the example above, the relevant ones are that the name can't have a space, apostrophe or a dash in it. So while, it might be more readable to us to have written `int employees mileage = 0;` or `int employee's-mileage = 0;`, those variations wouldn't be able to be translated. Instead, we compromise and name our variable in a way that we are able to read and in a way that the translator is able to translate, a.k.a `employeesMileage`.

> Ok, this was one part of the variable declaration that I could partially read before, but now is much more manageable. What about `int`? What does that stand for?

I mentioned before that "When a computer runs a program, the program needs to tell the computer when and how much space it needs to store values that it will need later on.". You were probably wondering how that is done. Possibly you even thought that is what the `0` does for us. If so, it most likely would have been very confusing as we stated that we want to store a value there and we are telling the computer how much space we need. If that were the case, it would be natural to ask, wouldn't we need more than zero space to store our value?

The reality is that there are a number of data types in c# that we can utilize in our programs. Each variable is a container, but the data type you use is what determines the size of that container and how the program understands the value stored in that container. `int` is a data type that is an abbreviation for integer. As our mileage will always be stored as full miles, an integer makes sense for this variable declaration.

Looking at the statement above now, we can understand it as the program telling the computer that we want an integer 'storage container' that we are going to refer to as `employeesMileage` to enable a person's ability to read the program.

___
To further illustrate this point:

Now imagine, the program we created for the CEO has been running for a year and he is very happy with it. He gave out golden wings and a bonus to the top three travelers in the company. The employees really liked it and it motivated them to be on the road a lot. The CEO has decided that he wants to tweak the program a bit. Rewarding only the top travelers for the year, provides a disincentive for newer employees. The CEO would also like to give out awards this year to those employees with the highest average miles traveled per month.

> How could we accomplish this?

The obvious first step would be to create another variable. We would do this in the same way we did above.

<pre>
<code>
int employeesMonthlyAverage = 0;
</code>
</pre>

Now imagine a scenario where at the end of the year, the company has employees with the following statistics.

- Employee1: 2 months, 5005 miles
- Employee2: 6 months, 15013 miles

When the program will run the math and store it to our new variable, the employeesMonthlyAverage for Employee1 will be 2502 and the employeesMonthlyAverage for Employee2 will be 2502.

**Wait, what?!? When I pull out my calculator, the monthly average for Employee1 should be 2502.5 and the monthly average for Employee2 should be 2502.166666.**

> What went wrong?

I stated before that when declaring a variable, "the data type you use is what determines the size of that container and how the program understands the value stored in that container". Ignoring the size aspect right now, the astute reader might already be able to spot our problem. When we were declaring the variable `employeesMonthlyAverage`, we told the program that that variable 'container' was reserved for storing an integer value.

I know that math class was a long time ago, so if integer is a familiar word, but the exact definition is escaping you, we can reference [Wikipedia](https://en.wikipedia.org/wiki/Integer) for a refresher. "An integer (from the Latin integer meaning "whole") is a number that can be written without a fractional component." So the program knows and enforces that the only values that it will store in the `employeesMonthlyAverage` variable will be integers. If you try to give it something else(like `2502.5` or `2502.166666`), it will store the integer part of the number(`2502`) and discard the rest.

> Great, I now see what is wrong, how can we fix it?

That is a great question and I suspect you know what the answer is, but might not know how to achieve the solution. If you are thinking that we need to use a different type, you are correct. Ok, so what types are there to choose from? There are many different types that exist out in the wild and we can even create our own(a topic for another time), but when we are starting out, it is best to start with the ['built-in' types](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/built-in-types-table). These types are a part of the language and will always be available to you when developing in c#.

Looking at the table, there are a number of different types. Some of their names might make apparent what can be stored there and others are not. For our purposes though, looking down the list, the first one that grabs my attention and seems to fit the bill would be `decimal`. Now to fix our program, we would update our `employeesMonthlyAverage` variable declaration to `decimal employeesMonthlyAverage = 0;` and now the program operates as we would expect.

> This is great! What else do I need to know about variables?

Variables is a large topic and there is a lot more we could discuss. For now though, lets look at a one more basic point about variables. As we stated for the purpose of the program, we are wanting to track an employee's mileage as they travel around on company business. We have discussed how we name a variable and how we tell the computer what size 'container' we need and how to tell the program how to interpret the value stored in that container.

> How do we actually store a value in that container though?

That can be accomplished through an 'assignment operator'. While we will get to operators in another article, for now, you can think of the `=` sign as the way you can tell the program to store a certain value in a certain container. For instance, when we are declaring the variable(as above), we can assign a value to it.

<pre>
<code>
decimal employeesMonthlyAverage = 2502.5;
</code>
</pre>

In this line of code:
 - We create a variable named `employeesMonthlyAverage`.
 - The program asks the computer for a container size that will hold a `decimal` value.
 - We tell the program to only allow `decimal` values to be stored in this variable's container.
 - We place the value(or 'assign' the value in programeese) `2502.5` into that container.

When the next year comes around and the average needs to be updated to reflect the current years average instead of last years, all we have to do is

<pre>
<code>
employeesMonthlyAverage = 5000;
</code>
</pre>

You will notice that the `decimal` is gone from our statement. At this point, we don't need to request another container from the computer or tell the program how to interpret the value stored in that container as that was done in the initial declaration of the variable. Now all we want to do is tell the computer to store a new value there. In this case that value is `5000`. Apparently our employee has been on the road much more this year, possibly due to the rewards program we have helped create for the company.

## **Success! Nice Work!!!**
