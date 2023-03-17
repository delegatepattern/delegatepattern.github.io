---
title: "How to Unit Test a Huge Codebase"
date: 2023-03-06T16:11:19-07:00
draft: false
---

Let's be honest. Unit Testing brings a physiological barrier when brought to the table of software developers. Personally, I find most developers don't fall in love with Unit Testing at all.

Since it is a hard topic, it requires a special approach to toggle.

I'll show you according to me some of the best ways to Unit Test a big codebase no matter how good or bad it is structured.

We have either one of the following types:
- 1: Codebase that is not testable (has no unit testing target or its poorly written with no testability mental modal in mind)
- 2: Codebase that is well structured, have some good unit tests coverage or you are about to write the whole app from scratch following best practices for unit testing.

## Bad Candiate Codebase

We have only a couple of options here:
- **Test Doubles, Mocks**: Test doubles allow you to unit test any class no matter how bad it is (you don't have to change any of the original class code). This method is complex, time consuming and gives really no benefit on the time invested. 

Why? Well, because the original class or codebase is poorly structured and not written in away to make it easily testable. These types of codes often change, and sometimes radically change over time. Thus, they cause your unit test to break, fail and require you to maintain it. This will take even more time every time the production code changes.

-  **Refactoring**: Here I mean, major restructuring of a codebase. This is a very time consuming and complex task. So, how we can achieve the best results without making such huge investment in time?

By splitting your codebase into seperate modules. This way, you treat and unit test each section in isolation of other parts of the code base. This allows you to fix more bugs and incrementally increase your unit testing coverage.

Modules can be created using Swift package Manager as a separate package for each part of your codebase. Example: You could have the Model layer in its own separate SP as an imported module.

## Good Candidate Codebase

You can still do the following:
- **Use Modules for specific parts of your code**: This still gives you huge benefit even if your good is testable from the first place. Big organizations use modules in their codebases because this allows them to speed up the development, assign code and component ownership to other teams in the origination and makes unit testing easier and faster.
- **Best Practices**: I won't go into the details. This post is not a tutorial or a complete guide, just to give you an overall all idea about how to toggle unit test in a large codebase. I would use the following techniques as best practices:

- 1: Utilizing values types over reference types whenever possible: If value types can be used instead of classes (no benefit of using reference types), then this makes unit testing cases straightforward. You basically avoid writing Test Doubles or Mocks that are much harder to write.
- 2: Avoiding using Singletons. Singletons gets into your own way when you perform unit or integration tests. You can use Dependency Injection instead.
- 3: Effectively dealing with reference and value types by separating the behavior and logic from your classes. Example:
If you have a model controller (most likely its reference types / class). You will find that this controller contains a couple of different parts:
-  A: **Behavior Code**: This is the reference type code that behaves and changes the state of the program. Anything related to the behavior or reference types can be extracted in its own methods. This will separate the reference code from value code.
-  B: **Logic Code**: Once you have extracted the code that behaves in its own methods, you will end up with the logic code that is nothing but value types (doesn’t send or receive messages or signals from the outside). Every class dose have some logic that is value type.

With this effective separation, you will be able to do the following easily:
- **Logic part testing**: Since this is value type, you can easily unit test it with value type view models. Straightforward.
- **Behavior part testing**: You ended up with little reference type code that easy to write, read and maintain. You might still need to write Test Double classes to test them, however, this significantly simplifies the testing techniques required. 

## Final Thought
Personaly, I use the 80/20 rule to statigically achieve quality and benfitial unit testing to my projects. 

Basically, you focus on the **RIGHT** 20% of your code to unit test (model layer and model controllers), that will generate 80% of the results for you. 

Don't let the high coverage number trick you. I can show you a sample class that have 100/100 unit testing coverage, but if you inspect it deeply and closely, you will see it misses some edge cases. So, how come?

Well, you will need to understand how Unit Testing Coverage is calculated by Xcode. 

It's based on the execution path of your methods. If a method (its line of code) gets executed, then this adds up to the total coverage. Not necessarily means you have covered all possible edge cases. So be careful when dealing with coverage as a whole number only.

I hope you find these insights helpful when you approach a new project that dose or dose not have a unit testing coverage.

Have questions or comments? feel free to send me an email. 






