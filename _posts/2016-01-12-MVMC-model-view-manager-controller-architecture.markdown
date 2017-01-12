---
layout: post
title: "MVMC: Model-View-Manager-Controller Architecture"
date: 2017-01-12 13:32:34
categories: projects
permalink: /mvmc-model-view-manager-controller-architecture/
excerpt: An architecture pattern for building REST APIs with room for reusable business-logic
---

I've built a lot of applications using the MVC architecture pattern. When an application starts getting messy, its pretty much what comes to the rescue, and once the layers are seperated, a lot of the code falls into place.

But I've been consistently seeing a pattern emerge out of all the projects I've built.

Firstly, the controller always gets messed up with code that modifies data into the form needed by a model's function. Like formatting dates, or parsing a parameter of comma seperated values into lists.

Secondly, many times, what a controller does is needed by another controller. This is especially true in REST APIs, where an API endpoint can depend on another API endpoint's functionality. There's no way this can be done convienently even if frameworks today offered this functionality, because in order to call the function, i'll still need to reformat all my data to the form which the controller expects it in.

Let me illustrate this with an example.

Lets say you have a listing function that filters and lists books based on various criteria. So you write an API endpoint called `/books/` which takes a bunch of filtering parameters and returns a list of books.

Now lets say you want to perform some operation on this selection. What do you do now? Well you'll have to replicate the API endpoint to filter everything similar to the `/books/` endpoint and then write additional functionality to perform the operation on those books.

There's no simple way to reuse the `/books/` endpoint to get you back the filtered list of items easily.

Of course, at this point, you can argue with me that you can write a function for filtering books which can be called by both APIs, but then, thats exactly what I"m getting at!

Instead of making the API endpoint controller handle the business logic, why not create a layer just below the controller to do it, freeing up the controller to handle the mundane tasks of formatting requests and responses?

This is exactly what I"m planning to do from now on. 

I'll build my models first, then a manager layer that handles all the business logic, which can call other managers and functions to get stuff done.

Then I'll build my controller which will focus on stuff like authenticating the requests, parsing parameters and so on.

Just to show off that I've come up with this (ahem!) brilliant plan, I'm giving myself a spot right next to MVC, and calling this architecture pattern, the Model-View-Manager-Controller architecture :P

Just kidding, I'm just trying to help fellow devs out here.

Give it a shot in your next project and let me know what you think.

I think you'll find it useful, because I've already used it in two projects and benefitted king-size ;)
