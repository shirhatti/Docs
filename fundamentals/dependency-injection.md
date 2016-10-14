---
uid: fundamentals/dependency-injection
---
<a name=fundamentals-dependency-injection></a>

  # Dependency Injection

[Steve Smith](http://ardalis.com), [Scott Addie](https://scottaddie.com)

ASP.NET Core is designed from the ground up to support and leverage dependency injection. ASP.NET Core applications can leverage built-in framework services by having them injected into methods in the Startup class, and application services can be configured for injection as well. The default services container provided by ASP.NET Core provides a minimal feature set and is not intended to replace other containers.

[View or download sample code](https://github.com/aspnet/Docs/tree/master/aspnet/fundamentals/dependency-injection/sample)

  ## What is Dependency Injection?

Dependency injection (DI) is a technique for achieving loose coupling between objects and their collaborators, or dependencies. Rather than directly instantiating collaborators, or using static references, the objects a class needs in order to perform its actions are provided to the class in some fashion. Most often, classes will declare their dependencies via their constructor, allowing them to follow the [Explicit Dependencies Principle](http://deviq.com/explicit-dependencies-principle/). This approach is known as "constructor injection".

When classes are designed with DI in mind, they are more loosely coupled because they do not have direct, hard-coded dependencies on their collaborators. This follows the [Dependency Inversion Principle](http://deviq.com/dependency-inversion-principle/), which states that *"high level modules should not depend on low level modules; both should depend on abstractions."* Instead of referencing specific implementations, classes, request abstractions (typically `interfaces`) which are provided to them when they are constructed. Extracting dependencies into interfaces and providing implementations of these interfaces as parameters is also an example of the [Strategy design pattern](http://deviq.com/strategy-design-pattern/).

When a system is designed to use DI, with many classes requesting their dependencies via their constructor (or properties), it's helpful to have a class dedicated to creating these classes with their associated dependencies. These classes are referred to as *containers*, or more specifically, [Inversion of Control (IoC)](http://deviq.com/inversion-of-control/) containers or Dependency Injection (DI) containers. A container is essentially a factory that is responsible for providing instances of types that are requested from it. If a given type has declared that it has dependencies, and the container has been configured to provide the dependency types, it will create the dependencies as part of creating the requested instance. In this way, complex dependency graphs can be provided to classes without the need for any hard-coded object construction. In addition to creating objects with their dependencies, containers typically manage object lifetimes within the application.

ASP.NET Core includes a simple built-in container (represented by the `IServiceProvider` interface) that supports constructor injection by default, and ASP.NET makes certain services available through DI. ASP.NET's container refers to the types it manages as *services*. Throughout the rest of this article, *services* will refer to types that are managed by ASP.NET Core's IoC container. You configure the built-in container's services in the `ConfigureServices` method in your application's `Startup` class.

Note: Martin Fowler has written an extensive article on [Inversion of Control Containers and the Dependency Injection Pattern](http://www.martinfowler.com/articles/injection.html). Microsoft Patterns and Practices also has a great description of [Dependency Injection](https://msdn.microsoft.com/en-us/library/dn178469(v=pandp.30).aspx).

Note: This article covers Dependency Injection as it applies to all ASP.NET applications. Dependency Injection within MVC controllers is covered in [Dependency Injection and Controllers](../mvc/controllers/dependency-injection.md).

  ## Using Framework-Provided Services

The `ConfigureServices` method in the `Startup` class is responsible for defining the services the application will use, including platform features like Entity Framework Core and ASP.NET Core MVC. Initially, the `IServiceCollection` provided to `ConfigureServices` has just a handful of services defined. Below is an example of how to add additional services to the container using a number of extension methods like `AddDbContext`, `AddIdentity`, and `AddMvc`.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [5, 8, 12], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/common/samples/WebApplication1/src/WebApplication1/Startup.cs"} -->

````c#

   // This method gets called by the runtime. Use this method to add services to the container.
   public void ConfigureServices(IServiceCollection services)
   {
       // Add framework services.
       services.AddDbContext<ApplicationDbContext>(options =>
           options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));

       services.AddIdentity<ApplicationUser, IdentityRole>()
           .AddEntityFrameworkStores<ApplicationDbContext>()
           .AddDefaultTokenProviders();

       services.AddMvc();

       // Add application services.
       services.AddTransient<IEmailSender, AuthMessageSender>();
       services.AddTransient<ISmsSender, AuthMessageSender>();
   }


   ````

The features and middleware provided by ASP.NET, such as MVC, follow a convention of using a single Add*Service* extension method to register all of the services required by that feature.

Tip: You can request certain framework-provided services within `Startup` methods through their parameter lists - see [Application Startup](startup.md) for more details.

Of course, in addition to configuring the application to take advantage of various framework features, you can also use `ConfigureServices` to configure your own application services.

  ## Registering Your Own Services

You can register your own application services as follows. The first generic type represents the type (typically an interface) that will be requested from the container. The second generic type represents the concrete type that will be instantiated by the container and used to fulfill such requests.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/common/samples/WebApplication1/src/WebApplication1/Startup.cs"} -->

````c#

   services.AddTransient<IEmailSender, AuthMessageSender>();
   services.AddTransient<ISmsSender, AuthMessageSender>();

   ````

Note: Each `services.Add<service>` call adds (and potentially configures) services. For example, `services.AddMvc()` adds the services MVC requires.

The `AddTransient` method is used to map abstract types to concrete services that are instantiated separately for every object that requires it. This is known as the service's *lifetime*, and additional lifetime options are described below. It is important to choose an appropriate lifetime for each of the services you register. Should a new instance of the service be provided to each class that requests it? Should one instance be used throughout a given web request? Or should a single instance be used for the lifetime of the application?

In the sample for this article, there is a simple controller that displays character names, called `CharactersController`. Its `Index` method displays the current list of characters that have been stored in the application, and initializes the collection with a handful of characters if none exist. Note that although this application uses Entity Framework Core and the `ApplicationDbContext` class for its persistence, none of that is apparent in the controller. Instead, the specific data access mechanism has been abstracted behind an interface, `ICharacterRepository`, which follows the [repository pattern](http://deviq.com/repository-pattern/). An instance of `ICharacterRepository` is requested via the constructor and assigned to a private field, which is then used to access characters as necessary.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [3, 5, 6, 7, 8, 14, 21, 23, 24, 25, 26], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/fundamentals/dependency-injection/sample/DependencyInjectionSample/Controllers/CharactersController.cs"} -->

````c#

   public class CharactersController : Controller
   {
       private readonly ICharacterRepository _characterRepository;

       public CharactersController(ICharacterRepository characterRepository)
       {
           _characterRepository = characterRepository;
       }

       // GET: /characters/
       public IActionResult Index()
       {
           PopulateCharactersIfNoneExist();
           var characters = _characterRepository.ListAll();

           return View(characters);
       }
       
       private void PopulateCharactersIfNoneExist()
       {
           if (!_characterRepository.ListAll().Any())
           {
               _characterRepository.Add(new Character("Darth Maul"));
               _characterRepository.Add(new Character("Darth Vader"));
               _characterRepository.Add(new Character("Yoda"));
               _characterRepository.Add(new Character("Mace Windu"));
           }
       }
   }

   ````

The `ICharacterRepository` simply defines the two methods the controller needs to work with `Character` instances.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [8, 9], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/fundamentals/dependency-injection/sample/DependencyInjectionSample/Interfaces/ICharacterRepository.cs"} -->

````c#

   using System.Collections.Generic;
   using DependencyInjectionSample.Models;

   namespace DependencyInjectionSample.Interfaces
   {
       public interface ICharacterRepository
       {
           IEnumerable<Character> ListAll();
           void Add(Character character);
       }
   }
   ````

This interface is in turn implemented by a concrete type, `CharacterRepository`, that is used at runtime.

Note: The way DI is used with the `CharacterRepository` class is a general model you can follow for all of your application services, not just in "repositories" or data access classes.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [9, 11, 12, 13, 14], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/fundamentals/dependency-injection/sample/DependencyInjectionSample/Models/CharacterRepository.cs"} -->

````c#

   using System.Collections.Generic;
   using System.Linq;
   using DependencyInjectionSample.Interfaces;

   namespace DependencyInjectionSample.Models
   {
       public class CharacterRepository : ICharacterRepository
       {
           private readonly ApplicationDbContext _dbContext;

           public CharacterRepository(ApplicationDbContext dbContext)
           {
               _dbContext = dbContext;
           }

           public IEnumerable<Character> ListAll()
           {
               return _dbContext.Characters.AsEnumerable();
           }

           public void Add(Character character)
           {
               _dbContext.Characters.Add(character);
               _dbContext.SaveChanges();
           }
       }
   }
   ````

Note that `CharacterRepository` requests an `ApplicationDbContext` in its constructor. It is not unusual for dependency injection to be used in a chained fashion like this, with each requested dependency in turn requesting its own dependencies. The container is responsible for resolving all of the dependencies in the graph and returning the fully resolved service.

Note: Creating the requested object, and all of the objects it requires, and all of the objects those require, is sometimes referred to as an *object graph*. Likewise, the collective set of dependencies that must be resolved is typically referred to as a *dependency tree* or *dependency graph*.

In this case, both `ICharacterRepository` and in turn `ApplicationDbContext` must be registered with the services container in `ConfigureServices` in `Startup`. `ApplicationDbContext` is configured with the call to the extension method `AddDbContext<T>`. The following code shows the registration of the `CharacterRepository` type.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [2, 3, 4, 10], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/fundamentals/dependency-injection/sample/DependencyInjectionSample/Startup.cs"} -->

````c#

   {
       services.AddDbContext<ApplicationDbContext>(options =>
           options.UseInMemoryDatabase()
       );

       // Add framework services.
       services.AddMvc();

       // Register application services.
       services.AddScoped<ICharacterRepository, CharacterRepository>();
       services.AddTransient<IOperationTransient, Operation>();
       services.AddScoped<IOperationScoped, Operation>();
       services.AddSingleton<IOperationSingleton, Operation>();
       services.AddSingleton<IOperationSingletonInstance>(new Operation(Guid.Empty));
       services.AddTransient<OperationService, OperationService>();
   }


   ````

Entity Framework contexts should be added to the services container using the `Scoped` lifetime. This is taken care of automatically if you use the helper methods as shown above. Repositories that will make use of Entity Framework should use the same lifetime.

Warning: The main danger to be wary of is resolving a `Scoped` service from a singleton. It's likely in such a case that the service will have incorrect state when processing subsequent requests.

  ## Service Lifetimes and Registration Options

ASP.NET services can be configured with the following lifetimes:

Transient
   Transient lifetime services are created each time they are requested. This lifetime works best for lightweight, stateless services.

Scoped
   Scoped lifetime services are created once per request.

Singleton
   Singleton lifetime services are created the first time they are requested (or when `ConfigureServices` is run if you specify an instance there) and then every subsequent request will use the same instance. If your application requires singleton behavior, allowing the services container to manage the service's lifetime is recommended instead of implementing the singleton design pattern and managing your object's lifetime in the class yourself.

Services can be registered with the container in several ways. We have already seen how to register a service implementation with a given type by specifying the concrete type to use. In addition, a factory can be specified, which will then be used to create the instance on demand. The third approach is to directly specify the instance of the type to use, in which case the container will never attempt to create an instance.

To demonstrate the difference between these lifetime and registration options, consider a simple interface that represents one or more tasks as an *operation* with a unique identifier, `OperationId`. Depending on how we configure the lifetime for this service, the container will provide either the same or different instances of the service to the requesting class. To make it clear which lifetime is being requested, we will create one type per lifetime option:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [5, 7], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/fundamentals/dependency-injection/sample/DependencyInjectionSample/Interfaces/IOperation.cs"} -->

````c#

   using System;

   namespace DependencyInjectionSample.Interfaces
   {
       public interface IOperation
       {
           Guid OperationId { get; }
       }

       public interface IOperationTransient : IOperation
       {
       }
       public interface IOperationScoped : IOperation
       {
       }
       public interface IOperationSingleton : IOperation
       {
       }
       public interface IOperationSingletonInstance : IOperation
       {
       }
   }
   ````

We implement these interfaces using a single class, `Operation`, that accepts a `Guid` in its constructor, or uses a new `Guid` if none is provided.

Next, in `ConfigureServices`, each type is added to the container according to its named lifetime:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/fundamentals/dependency-injection/sample/DependencyInjectionSample/Startup.cs"} -->

````c#

   services.AddScoped<ICharacterRepository, CharacterRepository>();
   services.AddTransient<IOperationTransient, Operation>();
   services.AddScoped<IOperationScoped, Operation>();
   services.AddSingleton<IOperationSingleton, Operation>();
   services.AddSingleton<IOperationSingletonInstance>(new Operation(Guid.Empty));
   services.AddTransient<OperationService, OperationService>();


   ````

Note that the `IOperationSingletonInstance` service is using a specific instance with a known ID of `Guid.Empty` so it will be clear when this type is in use. We have also registered an `OperationService` that depends on each of the other `Operation` types, so that it will be clear within a request whether this service is getting the same instance as the controller, or a new one, for each operation type. All this service does is expose its dependencies as properties, so they can be displayed in the view.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/fundamentals/dependency-injection/sample/DependencyInjectionSample/Services/OperationService.cs"} -->

````c#

   using DependencyInjectionSample.Interfaces;

   namespace DependencyInjectionSample.Services
   {
       public class OperationService
       {
           public IOperationTransient TransientOperation { get; }
           public IOperationScoped ScopedOperation { get; }
           public IOperationSingleton SingletonOperation { get; }
           public IOperationSingletonInstance SingletonInstanceOperation { get; }

           public OperationService(IOperationTransient transientOperation,
               IOperationScoped scopedOperation,
               IOperationSingleton singletonOperation,
               IOperationSingletonInstance instanceOperation)
           {
               TransientOperation = transientOperation;
               ScopedOperation = scopedOperation;
               SingletonOperation = singletonOperation;
               SingletonInstanceOperation = instanceOperation;
           }
       }
   }
   ````

To demonstrate the object lifetimes within and between separate individual requests to the application, the sample includes an `OperationsController` that requests each kind of `IOperation` type as well as an `OperationService`. The `Index` action then displays all of the controller's and service's `OperationId` values.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/fundamentals/dependency-injection/sample/DependencyInjectionSample/Controllers/OperationsController.cs"} -->

````c#

   using DependencyInjectionSample.Interfaces;
   using DependencyInjectionSample.Services;
   using Microsoft.AspNetCore.Mvc;

   namespace DependencyInjectionSample.Controllers
   {
       public class OperationsController : Controller
       {
           private readonly OperationService _operationService;
           private readonly IOperationTransient _transientOperation;
           private readonly IOperationScoped _scopedOperation;
           private readonly IOperationSingleton _singletonOperation;
           private readonly IOperationSingletonInstance _singletonInstanceOperation;

           public OperationsController(OperationService operationService,
               IOperationTransient transientOperation,
               IOperationScoped scopedOperation,
               IOperationSingleton singletonOperation,
               IOperationSingletonInstance singletonInstanceOperation)
           {
               _operationService = operationService;
               _transientOperation = transientOperation;
               _scopedOperation = scopedOperation;
               _singletonOperation = singletonOperation;
               _singletonInstanceOperation = singletonInstanceOperation;
           }

           public IActionResult Index()
           {
               // viewbag contains controller-requested services
               ViewBag.Transient = _transientOperation;
               ViewBag.Scoped = _scopedOperation;
               ViewBag.Singleton = _singletonOperation;
               ViewBag.SingletonInstance = _singletonInstanceOperation;
               
               // operation service has its own requested services
               ViewBag.Service = _operationService;
               return View();
           }
       }
   }
   ````

Now two separate requests are made to this controller action:

![image](dependency-injection/_static/lifetimes_request1.png)

![image](dependency-injection/_static/lifetimes_request2.png)

Observe which of the `OperationId` values varies within a request, and between requests.

* *Transient* objects are always different; a new instance is provided to every controller and every service.

* *Scoped* objects are the same within a request, but different across different requests

* *Singleton* objects are the same for every object and every request (regardless of whether an instance is provided in `ConfigureServices`)

  ## Request Services

The services available within an ASP.NET request from `HttpContext` are exposed through the `RequestServices` collection.

![image](dependency-injection/_static/request-services.png)

Request Services represent the services you configure and request as part of your application. When your objects specify dependencies, these are satisfied by the types found in `RequestServices`, not `ApplicationServices`.

Generally, you shouldn't use these properties directly, preferring instead to request the types your classes you require via your class's constructor, and letting the framework inject these dependencies. This yields classes that are easier to test (see [Testing](../testing/index.md)) and are more loosely coupled.

Note: Prefer requesting dependencies as constructor parameters to accessing the `RequestServices` collection.

  ## Designing Your Services For Dependency Injection

You should design your services to use dependency injection to get their collaborators. This means avoiding the use of stateful static method calls (which result in a code smell known as [static cling](http://deviq.com/static-cling/)) and the direct instantiation of dependent classes within your services. It may help to remember the phrase, [New is Glue](http://ardalis.com/new-is-glue), when choosing whether to instantiate a type or to request it via dependency injection. By following the [SOLID Principles of Object Oriented Design](http://deviq.com/solid/), your classes will naturally tend to be small, well-factored, and easily tested.

What if you find that your classes tend to have way too many dependencies being injected? This is generally a sign that your class is trying to do too much, and is probably violating SRP - the [Single Responsibility Principle](http://deviq.com/single-responsibility-principle/). See if you can refactor the class by moving some of its responsibilities into a new class. Keep in mind that your `Controller` classes should be focused on UI concerns, so business rules and data access implementation details should be kept in classes appropriate to these [separate concerns](http://deviq.com/separation-of-concerns/).

With regards to data access specifically, you can inject the `DbContext` into your controllers (assuming you've added EF to the services container in `ConfigureServices`). Some developers prefer to use a repository interface to the database rather than injecting the `DbContext` directly. Using an interface to encapsulate the data access logic in one place can minimize how many places you will have to change when your database changes.

<a name=replacing-the-default-services-container></a>

  ## Replacing the default services container

The built-in services container is meant to serve the basic needs of the framework and most consumer applications built on it. However, developers who wish to replace the built-in container with their preferred container can easily do so. The `ConfigureServices` method typically returns `void`, but if its signature is changed to return `IServiceProvider`, a different container can be configured and returned. There are many IOC containers available for .NET. In this example, the [Autofac](http://autofac.org/) package is used.

First, add the appropriate container package(s) to the dependencies property in `project.json`:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "javascript"} -->

````javascript

   "dependencies" : {
     "Autofac": "4.0.0",
     "Autofac.Extensions.DependencyInjection": "4.0.0"
   },
   ````

Next, configure the container in `ConfigureServices` and return an `IServiceProvider`:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [1, 11]}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#"} -->

````c#

   public IServiceProvider ConfigureServices(IServiceCollection services)
   {
     services.AddMvc();
     // add other framework services

     // Add Autofac
     var containerBuilder = new ContainerBuilder();
     containerBuilder.RegisterModule<DefaultModule>();
     containerBuilder.Populate(services);
     var container = containerBuilder.Build();
     return new AutofacServiceProvider(container);
   }
   ````

Note: When using a third-party DI container, you must change `ConfigureServices` so that it returns `IServiceProvider` instead of `void`.

Finally, configure Autofac as normal in `DefaultModule`:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#"} -->

````c#

   public class DefaultModule : Module
   {
     protected override void Load(ContainerBuilder builder)
     {
       builder.RegisterType<CharacterRepository>().As<ICharacterRepository>();
     }
   }
   ````

At runtime, Autofac will be used to resolve types and inject dependencies. [Learn more about using Autofac and ASP.NET Core](http://docs.autofac.org/en/latest/integration/aspnetcore.html).

  ## Recommendations

When working with dependency injection, keep the following recommendations in mind:

* DI is for objects that have complex dependencies. Controllers, services, adapters, and repositories are all examples of objects that might be added to DI.

* Avoid storing data and configuration directly in DI. For example, a user's shopping cart shouldn't typically be added to the services container. Configuration should use the [Options Model](configuration.md#options-config-objects.md). Similarly, avoid "data holder" objects that only exist to allow access to some other object. It's better to request the actual item needed via DI, if possible.

* Avoid static access to services.

* Avoid service location in your application code.

* Avoid static access to `HttpContext`.

Note: Like all sets of recommendations, you may encounter situations where ignoring one is required. We have found exceptions to be rare -- mostly very special cases within the framework itself.

Remember, dependency injection is an *alternative* to static/global object access patterns. You will not be able to realize the benefits of DI if you mix it with static object access.

  ## Additional Resources

* [Application Startup](startup.md)

* [Testing](../testing/index.md)

* [Writing Clean Code in ASP.NET Core with Dependency Injection (MSDN)](https://msdn.microsoft.com/en-us/magazine/mt703433.aspx)

* [Container-Managed Application Design, Prelude: Where does the Container Belong?](http://blogs.msdn.com/b/nblumhardt/archive/2008/12/27/container-managed-application-design-prelude-where-does-the-container-belong.aspx)

* [Explicit Dependencies Principle](http://deviq.com/explicit-dependencies-principle/)

* [Inversion of Control Containers and the Dependency Injection Pattern](http://www.martinfowler.com/articles/injection.html) (Fowler)
