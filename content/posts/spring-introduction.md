---
title: "Spring Boot: A quick introduction"
date: "2021-12-27T00:00:00"
disableShare: true
hideFooter: true
tags: ['java', 'spring', 'spring-boot']
categories: ['programming']
---

Five years ago I moved to [Spring Boot](https://spring.io/projects/spring-boot) from [ASP.NET](https://dotnet.microsoft.com/en-us/apps/aspnet). Frankly, I found Java and the Spring Framework confusing and it was not my choice to use it.

Since then I've used many other backend technologies (Ruby on Rails, ASP.NET Core, Express/TypeScript) but I keep coming back to Spring. I've really had a change of heart with Spring. 

Over those years I've had to teach many others about Spring and common questions come up:

- What exactly is Spring (and Spring Boot)
- Why is there so much 'magic' (beans) going on?
- Isn't Java a dead language?

> To clarify, I'm not here to convince anyone that "*Spring is awesome and suits every project*". But over the next few posts I **do** want to lower the bar for getting started.

## What **is** Spring?

Here's what the [Spring docs](https://docs.spring.io/spring-framework/docs/current/reference/html/overview.html#spring-introduction) have to say:

> Spring makes it easy to create Java enterprise applications. It provides everything you need to embrace the Java language in an enterprise environment, with support for Groovy and Kotlin as alternative languages on the JVM, and with the flexibility to create many kinds of architectures depending on an application’s needs.

That's fairly wordy but essentially its a framework for making enterprise applications. It can be further extended with [Modules](https://docs.spring.io/spring-framework/docs/3.0.0.M4/reference/html/ch01s02.html) which are community driven. When people say "Spring" they refer to not just the Spring Framework but also to the modules built on top of it - the Spring stack or ecosystem.

This has the advantage of having many options but can also be very frustrating to newcomers.

### What does Spring mean by 'Enterprise'?

I'm not sure why Spring insists on name dropping enterprise as it can turn off many smaller developers and make the framework appear 'heavy'.

I would assert that many common enterprise problems are problems anyone faces when taking a project from their laptop to production:

- **Support multiple deployment environments**. In enterprise land this means not having direct access to production. Even if you do, you shouldn't.
- **Dependency injection and inversion of control**. In the world of enterprise this can be different LDAP implementations, but even a local developer benefits by having a mock third party integration when running locally vs a real one in production.
- **Transaction management**. Many enterprises demand proper transactions around SQL updates, this is just good practice at any scale.

### Who owns Spring?

The framework is entirely [Open Source](https://github.com/spring-projects/spring-framework) and has an [Apache 2.0 license](https://tldrlegal.com/license/apache-license-2.0-(apache-2.0)). It is 'free' as in beer.

From an ownership perspective, the original founders' company  - Spring (previously SpringSource) was brought by [Pivitol Software](https://en.wikipedia.org/wiki/Pivotal_Software) which it self is now [VMware Tanzu Labs](https://tanzu.vmware.com/content/blog/vmware-tanzu-labs-new-name).

Looking at the [contributors graph](https://github.com/spring-projects/spring-framework/graphs/contributors) for Spring you'll see VMware/Pivitol staff as the main core contributors.

> *TLDR; Its open source but VMware owns the trademark.*

### What is Spring Boot

I hear people describe Spring Boot like this: 

> Spring Boot is a bootstrap project (hence the name) maintained by the Spring team. It has sensible defaults already set and is designed to get going as quickly as possible.

And I mostly agree. Its similar to [Ruby on Rails](https://rubyonrails.org/) or [Create React App](https://reactjs.org/docs/create-a-new-react-app.html) in that it gives you a stack to get going quickly.

Typically people will use the [Spring Initializr](https://start.spring.io/) and choose additional dependencies such as [Spring Web](https://search.maven.org/artifact/org.springframework/spring-web) for a RESTful service with built in [Apache Tomcat Server](https://tomcat.apache.org/) and [Spring Security](https://spring.io/projects/spring-security) for authorization and authentication.

This is all nice, but in reality the biggest advantages when using Spring Boot is:

- All projects have a familiar structure.
- Common tasks (such as setting up logging, IoC, Maven, environment settings, etc) are setup saving time.

The second point is *nice*, but the first should be repeated:

> A common opinionated view way of working is a game changer when choosing a technology and forming a team.

### What Spring isn't

I would assert that a project (aka module) being in the Spring Framework is *not* any kind of seal of quality. Some projects are far more popular and well funded than others. 

It's not impossible for projects to [leave](https://spring.io/blog/2018/12/12/spring-cloud-greenwich-rc1-available-now#spring-cloud-netflix-projects-entering-maintenance-mode) the framework. 

Remember, it's a collection of projects and modules. It's down to experience to pick the ones right for you. Spring is only heavy if you want it to be.