---
layout: post
title: "Dependency Injection and Using Guice"
author: "Jiahui"
categories: blog
tags: [blog]
image: morpho-1.jpg
---

# Dependency Injection and Using Guice

## Dependency Injection
Simply put... A client receives service instances that it dependents on.

```java
YourService service = new YourServiceImpl(new ServiceA(new ServiceB(new ServiceC())));
```

[For Details: https://en.wikipedia.org/wiki/Dependency_injection]


## Using Guice for Dependency Injection
[Guice](https://github.com/google/guice/), a simple DI framework for Java. 

As its Wiki writes, you can think of Guice as a map to bind each dependency to a provider.

### Main purpose
Instead of letting clients to look up dependencies as the above pattern, an injector interface will handle that. (`@Inject` is the new `A = new B();`). This interface should be defined in a module class, binding interfaces or class with the implementation or provider you want, and using injectors to create new instances according to your bindings.

### An Example
We have this old-fashioned BugGeneratorService:
```java
interface BugGeneratorService {
    public Bug generateBug();
}

class RandomBugGeneratorService implements BugGeneratorService {
    BugDao bugDao; // Data accessor to get a bug instance
    
    // Constructor
    RandomBugGeneratorService(BugDao bugDao) {
        this.bugDao = bugDao;
    }

    // method to generate a random bug
    @Override
    public Bug generateBug() {
        Bug bug = bugDao.getRandomBug();
        return bug;
    }
}
```

A client will do something like this, or put it inside a factory class.
```java
BugDao bugDao = new NetworkBugDao();
BugGeneratorService service = new RandomBugGeneratorService(bugDao);
service.generateBug();
```

#### Moving to Guice
Define Module to declare dependencies. Here, BugGeneratorService is bound to RandomBugGeneratorService and BugDao is bound to a provider for NetworkBugDao.

```java
class MyModule extends AbstractModule {
    @Override
    protected void configure() {
        bind(BugGeneratorService.class).to(RandomBugGeneratorService.class);
        
        // Assume if we already have a NetworkBugDaoProvider class
        bind(BugDao.class).toProvider(NetworkBugDaoProvider.class);
        // if we need a new provider class
        @Provides
        BugDao provideBugDao() {
            return NetworkBugDao();
        }
    }
}
```

Use Constructor Injection in service implementation. The `@Inject` annotation will let Guice look for instance for BugDao, in this case it will trigger NetworkBugDaoProvider when creating service with Guice.
```java
class RandomBugGeneratorService implements BugGeneratorService {
    BugDao bugDao;
    
    // Constructor
    @Inject
    RandomBugGeneratorService(BugDao bugDao) {
        this.bugDao = bugDao;
    }

    @Override
    public Bug generateBug() {
        Bug bug = bugDao.getRandomBug();
        return bug;
    }
}
```

Then when you want to use BugGeneratorService
```java
Injector injector = Guice.createInjector(new MyModule());
BugGeneratorService bugGeneratorService = injector.getInstance(BugGeneratorService.class);
service.generateBug();
```

Write unit tests as usual using constructor to create instance
```java
class RandomBugGeneratorServiceUnitTest {
    @Mock
    BugDao dao;

    @Before
    void Setup() {
        RandomBugGeneratorService service = new RandomBugGeneratorService(dao);
    }
    // unit tests...
}
```

Integration tests with injector. The `@Inject` annotation will get RandomBugGeneratorService instance for BugGeneratorService, avoiding redundant code for creating RandomBugGeneratorService with BugDao.
```java
class BugGeneratorServiceIntegrationTest {
    @Inject
    BugGeneratorService service;

    // tests...
}
```

### More Examples
Besides what we have talked about, there are more things you can make use of.
```java
// Applying Scopes to reuse the instance created at the first time 
bind(BugGeneratorService.class).to(RandomBugGeneratorService.class).asEagerSingleton();
bind(BugGeneratorService.class).to(RandomBugGeneratorService.class).in(Singleton.class);
// with @Singleton when doing @Provides
@Provides @Singleton
BugDao provideBugDao() {
    return NetworkBugDao();
}
```

### Common Question
#### injecting constructor or fields?
Injecting constructor is the recommended way when you're using final objects. Plus it is easy to write tests using this design.

Injecting fields is neither testable or safe to use for immutable field.
#### want a constructor with parameters from the caller
```java
class RandomBugGeneratorService implements BugGeneratorService {
    // Constructor
    RandomBugGeneratorService(BugDao bugDao, Date date, int bugNumber) {
        // ...
    }
}
```

Solution #0: add a factory to create instance only with a method to create new instance with parameters from caller, while injecting from Guice when creating the factory.
```java
interface BugGeneratorServiceFactory {
    create(Date date, int bugNumber);
}

class RandomBugGeneratorServiceFactory implements BugGeneratorServiceFactory {
    BugDaoProvider bugDaoProvider;

    @Inject
    RandomBugGeneratorServiceFactory(BugDaoProvider bugDaoProvider) {
        this.bugDaoProvider = bugDaoProvider;
    }

    create(Date date, int bugNumber) {
        return RandomBugGeneratorService(bugDaoProvider.get(), date, bugNumber);
    }
}
```
and bind BugGeneratorServiceFactory to RandomBugGeneratorServiceFactory.

Solution #1: AssistedInject

we still need factories to handle parameters from Guice and caller separately, but use `@Assisted` to avoid manually writing RandomBugGeneratorServiceFactory class in solution#1. It will link parameters with `@Assisted` to the `create()` method in factory, and bind the base factory(BugGeneratorServiceFactory) to the provider generated by Guice.
```java
class RandomBugGeneratorService implements BugGeneratorService {
    @Inject
    RandomBugGeneratorService(BugDao bugDao, @Assisted Date date, @Assisted int bugNumber) {
        // ...
    }
}
```

```java
install(new FactoryModuleBuilder()
     .implement(BugGeneratorService.class, RandomBugGeneratorService.class)
     .build(BugGeneratorServiceFactory.class));
```