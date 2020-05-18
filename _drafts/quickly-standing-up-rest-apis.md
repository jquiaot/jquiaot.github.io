---
layout: post
title:  "Quickly Standing Up REST APIs"
date:   2020-05-09 17:00:00 -0700
categories: software-development rest-api python java golang
---
During a recent interview, I was given a take-home coding exercise to create a simple REST API providing some bit of functionality. The use cases for the API were pretty simple, and it was expected that the exercise shouldn't take more than 2 hours. I won't go into the specifics of this particular interview exercise, but it did have me scrambling a little bit, as it had been a while since I created a REST API completely from scratch. Fortunately, I had been working with Spring Boot recently, so I opted to create the API that way. I did manage to code up the whole thing in two hours, but it got me thinking - how would I do this in other languages? Are there quicker ways to do this?

This post aims to demonstrate some quick ways to create a REST API from scratch, using different languages. I'd like to leave at home the debate over how efficient each of these ways might be, how correct the code is, etc. My primary purpose is to create, from scratch, some minimally-viable REST API that can handle some rudimentary operations. Further posts will hopefully explore other aspects of this task, including but not limited to: using some persistent store; automated builds and tests; containerization; etc.

# Java/Spring Boot

I chose a Java-based Spring Boot application as my approach because I felt most comfortable with the Java idioms and conventions. Java itself has some low-level constructs that you can use to e.g. handle requests, spawn workers, etc., but for the most part people work within a framework that gives us all of these for "free", so that they only have to focus (more or less) on the business logic. Spring is a popular and mature framework suitable for many different kinds of applications. Spring Boot is a very opinionated Spring-based framework for creating microservices.

## Bootstrapping the Project

Spring provides the Spring Initializr tool to generate a Maven- or Gradle-based Spring Boot web application project, saving you the time of creating the proper directories, the initial `pom.xml` file, etc. You just visit `https://start.spring.io/`, specify your project parameters, and hit the "Generate" button to generate a zip file that is your base project. Simply unpack the zip file into your working directory, and off you go.

## Creating the Resource Class

The Spring convention is to create Java classes to encapsulate the resources for your REST API.

## Mapping Requests to Controller Methods

The Spring convention is to create Java classes annotated with `@Controller` to indicate to the Spring runtime that that class contains methods for handling HTTP requests.


## Building and Running

# Python

I've been brushing up on my Python skills of late. Python also is a popular language for developing web applications, and there are a few frameworks/modules that most people end up using: Flask and Django. Flask, as I understand it, is a smaller and more lightweight framework for building web applications, whereas Django is more feature-filled and preferred by folks who are building larger or more complex applications. I chose Flask for this task because of its claim at being more lightweight.

## Bootstrapping the Project

# Golang

Go (Golang) is a relatively newer language, albeit one that has been gaining significant popularity of late. I guess it helps that Go was designed at Google and has heavyweights like Ken Thompson and Rob Pike as its creators. Go is "syntactically similar to C, but with memory safety, garbage collection, structural typing,[6] and CSP-style concurrency."

## Bootstrapping the Project


# Conclusions

It seems with all modern languages, there are ways and means to quickly bootstrap and create a web application or REST API.

# Next Steps

Standing up a REST API is the first step, but what's next?

* Containerization
* CI/CD
