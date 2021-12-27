---
title: "Spring Boot: Magic Beans"
disableShare: true
hideFooter: true
draft: true
---

A common complaint about [Spring](https://spring.io) refers to *Spring Magic* and general confusion about how dependency injection works in Spring. It can be extremely confusing to get a stacktrace like this:

```bash
Error creating bean with name 'beanA': Injection of autowired dependencies failed; 
nested exception is org.springframework.beans.factory.BeanCreationException: 
Could not autowire field: private com.example.demo.BeanB com.example.demo.BeanA.dependency; 
nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException: 
No qualifying bean of type [com.example.demo.BeanB] found for dependency: 
expected at least 1 bean which qualifies as autowire candidate for this dependency. 
Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}
```

## The challenges of Inversion of Control

[Inversion of Control (IoC)](https://en.wikipedia.org/wiki/Inversion_of_control) is a way to create an object without expressly creating or managing its dependencies. 

To explain why you may want IoC, I'm going to use a common [onion architecture](https://en.everybodywiki.com/Onion_Architecture) example seen in many REST services.

{{< mermaid align="left" theme="neutral" >}}
flowchart LR
    RestController --> BusinessService --> PersistenceService -.-> Database[(Database)]
    RestController -->  AnotherBusinessService --> PersistenceService 
{{< /mermaid >}}

Lets say we want to make a new instance of a `RestController`: 

```Java
public class RestController {
    final BusinessService businessService;
    final AnotherBusinessService anotherBusinessService;

    public RestController() {
        var persistenceService = new PersistenceService("someDatabaseConnection?");
        businessService = new BusinessService(persistenceService);
        anotherBusinessService = new AnotherBusinessService(persistenceService);
    }
}
```

Thats.. fairly messy. We can imagine that as `BusinessService` expands to need more of its own dependencies we're quickly going to get into maintainability issues.

- Where would you instantiate a common singleton across many controllers?
- What happens if PersistenceService has more of its *own* dependencies? Suddenly your passing dependencies of dependencies down many layers.
- As your application gets larger, you can imagine needing logging classes, database connections, etc. Thats a lot of crud in your constructors.
- This constructor logic can pollute downstream classes, violating the [Single Responsibility Principle](https://en.wikipedia.org/wiki/Single-responsibility_principle).
- Managing your own dependencies can cause [circular dependency issues](https://dotnetcoretutorials.com/2020/09/14/dealing-with-circular-dependency-injection-references/).

To give another example, imagine a scenario where you wanted a certain Database in production (MySQL) but a memory database locally (H2). In this circumstance you would need to put a nasty if somewhere in your code, depending on the environment. 

Its not.. *bad* but its not nice either. Lets see what other options we have available to us.

## So what is a bean?

A *Bean* is a Spring concept for an object that are under dependency management by the Spring Framework. Spring uses its own [IoC Container](https://docs.spring.io/spring-framework/docs/3.2.x/spring-framework-reference/html/beans.html) that does this heavy lifting for you.

To register a bean you need to let Spring know **what** it should be managing and **how** it should be managed. In our example `BusinessService`, `AnotherBusinessService` and `PersistenceService` all should be managed. To do that, we need to use an `Annotation`.

```Java
@Service // Added Spring annotation
public class RestController {
   
}
```

By adding the [@Service](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Service.html) annotation we're telling Spring that this class is a `Business Service Facade` aka its a business logic class and will be reused.

After adding that annotation to the other classes, we can refactor our `RestController`.

```Java
public class RestController {
    @Autowired // Added Spring annotation
    final BusinessService businessService;
    @Autowired // Added Spring annotation
    final AnotherBusinessService anotherBusinessService;

    // Constructor has been removed.
}
```

We've added [@Autowired](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/annotation/Autowired.html) and removed the constructor. I hope you will agree that this is much cleaner and ultimately maintainable. But how does `Autowired` work? 

When the application starts, Spring looks for usages of `@Autowired`. Then it looks for any registered beans, in our case they have been registered using `@Service`. It will then consolidate all dependencies to reuse as many as possible during its runtime.

> There are other annotations that could be used (`@Component` or `@Repository`) but `@Service` seems to most appropriate in this use case. 
> 
> *Correct annotations can be a good source for documentation.*
 
I hope you agree that an Inversion of control framework can make code much more sustainable and ultimately more stable.

### An alternative

Some people do not like this style of injection as they feel using `@Autowired` too much like 'magic' and not close enough to regular Java. 

I suspect this comes down to *"Are you okay with Annotations in Java?"*.

While Spring is very annotation heavy, the Spring team is now offering (and even [recommending](https://docs.spring.io/spring-framework/docs/4.2.x/spring-framework-reference/html/beans.html)) constructor injection. This is far closer to regular Java and other languages:

```Java
public class RestController {
    final BusinessService businessService;
    final AnotherBusinessService anotherBusinessService;

    public RestController(BusinessService businessService, AnotherBusinessService anotherBusinessService) {
        this.businessService = businessService;
        this.anotherBusinessService = anotherBusinessService;
    }
}
```

Effectively Spring just 'knows' about what a `BusinessService` is from the `@Service` annotation. This keeps the RestController clean and understandable, it also gives us flexibility by still having a constructor.

I personally do prefer this style as it removes two annotations and makes the code a bit easier to understand. We also have a constructor we can now use in our unit tests if required.

### When it goes wrong

Theres a few things to look out for when it comes to Spring IoC.

#### Don't use new

Any attempts to create an object manually will mean Spring ignores that object and all its descendants. This may seem obvious but effectively when you annotate a class with `@Service`, `@Component` or `@Repository` you are handing over ownership to Spring IoC to create and manage that class. Using the `new` keyword will cause errors as the objects dependency tree wont be properly created.

Its very much *all or nothing* when it comes to Spring IoC. In certain circumstances this fine (such as unit testing) but overall, its best to avoid creating objects yourself and letting Spring handle it.

#### You've added another class to a constructor

Using our example from above, if you add another class to the constructor but do not annotate with `@Service` you will see an error:

```Java
public class RestController {
    final BusinessService businessService;
    final AnotherBusinessService anotherBusinessService;
    final NewService newService;

    public RestController(BusinessService businessService, AnotherBusinessService anotherBusinessService, NewService newService) {
        this.businessService = businessService;
        this.anotherBusinessService = anotherBusinessService;
        this.newService = newService; // new service without @Service annotation!
    }
}

// No annotation
public class NewService() {}
```

`NewService` is not known...

```java
Error creating bean with name 'NewService': Injection of autowired dependencies failed; 
nested exception is org.springframework.beans.factory.BeanCreationException: 
Could not autowire field: private com.example.demo.NewService com.example.demo.NewService.dependency; 
nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException: 
No qualifying bean of type [com.example.demo.NewService] found for dependency: 
expected at least 1 bean which qualifies as autowire candidate for this dependency. 
Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}
```

In this situation adding `@Service` to `NewService` would register the class and allow Spring to manage this bean correctly.

```java
@Service // Annotation added. Bean should now be registered.
public class NewService() {}
```

*The above error stacktrace is the same example from the introduction. I hope this makes feels more approachable now!*

### Using alternative services

In this final example we want to vary which service we get depending on if we're running locally or in production. This is exceptionally powerful.

{{< mermaid align="left" theme="neutral" >}}
flowchart LR
    RestController --> BusinessService -->  LocalBusinessServiceImpl
    RestController --> BusinessService --> ProductionBusinessServiceImpl
{{< /mermaid >}}

First we create an interface and each service with the `@Service` annotation to let Spring know about them.

```Java
public interface BusinessService {
    String getEnvironment();
}

@Service
public class LocalBusinessServiceImpl implements BusinessService {
    public String getEnvironment() {
        return "Local";
    }
}

@Service
public class ProductionBusinessServiceImpl implements BusinessService {
    public String getEnvironment() {
        return "Production";
    }
}

public class RestController {
    final BusinessService businessService;

    public RestController(BusinessService businessService) {
        this.businessService = businessService;
    }
}
```

Now we have an issue if we start the application Spring will throw a stacktrace error similar to:

```bash
Description:

Field businessService in com.example.demo.RestController required a single bean, but 2 were found:
    - businessService: defined by method 'businessService' in class path resource [com/example/LocalBusinessServiceImpl.class]
    - businessService: defined by method 'businessService' in class path resource [com/example/ProductionBusinessServiceImpl.class]

Action:

Consider marking one of the beans as @Primary, updating the consumer to accept multiple beans, or using @Qualifier to identify the bean that should be consumed
```

Newcomers can get frustrated by the word `bean` and while the action appears clear, it does not help the programmer make a decision on where or why to add [@Primary](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Primary.html).

Whats happening is Spring says *"You have two possible candidates, which do you want to use?"*. Remember that Spring IoC needs to know not only *what* but *when* to use a Bean.

Lets fix our code:

```Java
// Interface and RestController unchanged.

@Profile("dev") // New annotation
@Service
public class LocalBusinessServiceImpl implements BusinessService {
    public String getEnvironment() {
        return "Local";
    }
}

@Profile("prod") // New annotation
@Service
public class ProductionBusinessServiceImpl implements BusinessService {
    public String getEnvironment() {
        return "Production";
    }
}
```

Here we've added a [@Profile](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Profile.html) annotation to tell Spring *when* to use each bean.

We also could have also used one of the recommended actions from the previous stacktrace:

```Java
// Interface, RestController unchanged.

@Service
public class LocalBusinessServiceImpl implements BusinessService {
    public String getEnvironment() {
        return "Local";
    }
}

@Profile("!dev") // If its not dev
@Order(1) // Use this one first
@Service
public class ProductionBusinessServiceImpl implements BusinessService {
    public String getEnvironment() {
        return "Production";
    }
}
```

> Remember, if you see a 'bean' error its because Spring either has **too many** options or **no** options when configuring IoC.

## Conclusion

I hope after this introduction to Spring you understand a bit more about:

- Why you should be using Spring's inversion of control.
- How to configure a Spring Bean
- Potential methods for resolving Spring bean stacktraces.
