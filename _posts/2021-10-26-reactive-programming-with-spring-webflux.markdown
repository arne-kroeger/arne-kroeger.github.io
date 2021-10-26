---
layout: single
title:  "Spring GraphQL server based on webflux - Part 1 - Intro"
date:   2021-10-26 19:13:49 +0200
categories: java spring
comments: true
tags: java spring webflux graphql reactive
---

## Reactive programming

Reactive programming is a programming paradigm and has several implementation especially in the frontend world (e.g. [VueJS](https://vuejs.org/){:rel="nofollow"}, [Angular](https://angular.io/){:rel="nofollow"}, [React](https://reactjs.org/){:rel="nofollow"}). In the backend technologies it is not so widely used but could help for complex systems with huge amounts of parallel requests or many sources data changes.

For more details on reactive programming see:

- [What is Reactive Programming? by Keval Patels](https://medium.com/@kevalpatel2106/what-is-reactive-programming-da37c1611382){:rel="nofollow"}
- [Reactive Spring by Josh Long](https://reactivespring.io/){:rel="nofollow"}
- [Is Spring WebFlux a Myth? by Gavin Fong](https://blog.devgenius.io/is-spring-webflux-a-myth-4526c2f92413){:rel="nofollow"}

It is not useful in all use-case and could possible add more complexity to your projects. You should check your use-case if it fits into it. For more details see the following blog posts:

- [Reactive Programming: Why, What and When? by Mohit Snehal](https://blog.devgenius.io/reactive-programming-why-what-and-when-e00495cda9c4){:rel="nofollow"}
- [Java Reactive Programming - Effective Usage in a Real World Application by Stefan Nothaas](https://tech.trivago.com/2021/03/16/java-reactive-programming-effective-usage-in-a-real-world-application/){:rel="nofollow"}

For me it has been a nice to know use-case and was necessary to evaluate it as an option for future developments. 

### Spring implementation

Spring added a webflux component to implement a reactive alternative to Spring MVC (based on the [Reactor project](https://projectreactor.io/){:rel="nofollow"}). 

For more details see [Spring reactive project page](https://spring.io/reactive){:rel="nofollow"}.

## Graphql

I'm using Spring most of the time as my services backend and I've used [OpenAPI](https://www.openapis.org/){:rel="nofollow"} based REST services several times and started some projects with a GraphQL. As Spring started the GraphQL server project based on webflux I want to show this sample implementation to deliver a starting point.

For more general purpose and pros / cons of GraphQL see the following links:

- [GraphQL Website](https://graphql.org/){:rel="nofollow"}
- [GraphQL vs. REST: A Comprehensive Comparison by ALFRICK OPIDI ](https://blog.api.rakuten.net/graphql-vs-rest/){:rel="nofollow"}
- [Spring GraphQL documentation](https://docs.spring.io/spring-graphql/docs/1.0.0-SNAPSHOT/reference/html/){:rel="nofollow"}

## Project description

> The sample project created in this description bases on the code examples provided by Josh Long in his video [Spring Tips: Spring GraphQL](https://www.youtube.com/watch?v=kVSYVhmvNCI&ab_channel=SpringDeveloper){:rel="nofollow"}

We want to build a simple GraphQL service which is providing company and person data via a GraphQL service build on top of a Spring GraphQL webflux backend. For this example project I use a classical RDBMS system (PostgreSQL) as it is used in many of my projects as a really reliable, scalable and extendible data source with a huge amount of features / possibilities. 

### The problem with the JDBC driver in the reactive world

The JDBC driver is a problem in terms of a reactive project because of its IO blocking implementation. The driver will block all further processing unless all data has been fetched from the datasource. This problem can be covered with the R2DBC (Reactive Relational Database Connectivity) driver which allows a reactive connection handling with classical RDBMS like PostgreSQL, MySQL, MariaDB, MS SQL Server, Oracle and H2. 

For more details see:

- [R2DBC Resources about the project](https://r2dbc.io/resources){:rel="nofollow"}
- [R2DBC vs. JDBC Performance comparison by Maarten Smeets](https://technology.amis.nl/software-development/performance-and-tuning/performance-of-relational-database-drivers-r2dbc-vs-jdbc/){:rel="nofollow"}
- [Spring: Blocking vs non-blocking: R2DBC vs JDBC and WebFlux vs Web MVCby Maarten Smeets](https://technology.amis.nl/software-development/performance-and-tuning/spring-blocking-vs-non-blocking-r2dbc-vs-jdbc-and-webflux-vs-web-mvc/){:rel="nofollow"}

### Summary 

So the tech stack for this sample project will be:

- Spring Boot
- Spring GraphQL
- Spring WebFlux
- R2DBC
- PostgreSQL 

As this project will grow we will add some new items to this list from chapter to chapter to cover more things that will cross my mind.

In Part 2 we will setup the project to make our first GraphQL requests to the database.