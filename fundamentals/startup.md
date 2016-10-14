---
uid: fundamentals/startup
---
Warning: This page documents version 1.0.0-rc1 and has not yet been updated for version 1.0.0

<a name=application-startup></a>

  # Application Startup

By [Steve Smith](http://ardalis.com)

ASP.NET Core provides complete control of how individual requests are handled by your application. The `Startup` class is the entry point to the application, setting up configuration and wiring up services the application will use. Developers configure a request pipeline in the `Startup` class that is used to handle all requests made to the application.

  ## The Startup class

In ASP.NET Core, the `Startup` class provides the entry point for an application, and is required for all applications. It's possible to have environment-specific startup classes and methods (see [Working with Multiple Environments](environments.md)), but regardless, one `Startup` class will serve as the entry point for the application. ASP.NET searches the primary assembly for a class named `Startup` (in any namespace). You can specify a different assembly to search using the *Hosting:Application* configuration key. It doesn't matter whether the class is defined as `public`; ASP.NET will still load it if it conforms to the naming convention. If there are multiple `Startup` classes, this will not trigger an exception. ASP.NET will select one based on its namespace (matching the project's root namespace first, otherwise using the class in the alphabetically first namespace).

The `Startup` class can optionally accept dependencies in its constructor that are provided through [dependency injection](dependency-injection.md).  Typically, the way an application will be configured is defined within its Startup class's constructor (see [Configuration](configuration.md)). The Startup class must define a `Configure` method, and may optionally also define a `ConfigureServices` method, which will be called when the application is started.

  ## The Configure method

The `Configure` method is used to specify how the ASP.NET application will respond to individual HTTP requests. At its simplest, you can configure every request to receive the same response. However, most real-world applications require more functionality than this. More complex sets of pipeline configuration can be encapsulated in [middleware](middleware.md) and added using extension methods on [IApplicationBuilder](https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Builder/IApplicationBuilder/index.html).

Your `Configure` method must accept an [IApplicationBuilder](https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Builder/IApplicationBuilder/index.html) parameter. Additional services, like `IHostingEnvironment` and `ILoggerFactory` may also be specified, in which case these services will be [injected](dependency-injection.md) by the server if they are available. In the following example from the default web site template, you can see several extension methods are used to configure the pipeline with support for [BrowserLink](http://www.asp.net/visual-studio/overview/2013/using-browser-link), error pages, static files, ASP.NET MVC, and Identity.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [8, 9, 10, 14, 17, 19, 23], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": true, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/common/samples/WebApplication1/src/WebApplication1/Startup.cs"} -->

````c#

   public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
   {
       loggerFactory.AddConsole(Configuration.GetSection("Logging"));
       loggerFactory.AddDebug();

       if (env.IsDevelopment())
       {
           app.UseDeveloperExceptionPage();
           app.UseDatabaseErrorPage();
           app.UseBrowserLink();
       }
       else
       {
           app.UseExceptionHandler("/Home/Error");
       }

       app.UseStaticFiles();

       app.UseIdentity();

       // Add external authentication middleware below. To configure them please see http://go.microsoft.com/fwlink/?LinkID=532715

       app.UseMvc(routes =>
       {
           routes.MapRoute(
               name: "default",
               template: "{controller=Home}/{action=Index}/{id?}");
       });
   }

   ````

Each `Use` extension method adds [middleware](middleware.md) to the request pipeline. For instance, the `UseMvc` extension method adds the [routing](routing.md) middleware to the request pipeline and configures [MVC](../mvc/index.md) as the default handler.

You can learn all about middleware and using [IApplicationBuilder](https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Builder/IApplicationBuilder/index.html) to define your request pipeline in the [Middleware](middleware.md) topic.

  ## The ConfigureServices method

Your `Startup` class can optionally include a `ConfigureServices` method for configuring services that are used by your application. The `ConfigureServices` method is a public method on your `Startup` class that takes an [IServiceCollection](https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/Extensions/DependencyInjection/IServiceCollection/index.html) instance as a parameter and optionally returns an `IServiceProvider`. The `ConfigureServices` method is called before `Configure`. This is important, because some features like ASP.NET MVC require certain services to be added in `ConfigureServices` before they can be wired up to the request pipeline.

Just as with `Configure`, it is recommended that features that require substantial setup within `ConfigureServices` be wrapped up in extension methods on [IServiceCollection](https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/Extensions/DependencyInjection/IServiceCollection/index.html). You can see in this example from the default web site template that several `Add[Something]` extension methods are used to configure the app to use services from Entity Framework, Identity, and MVC:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [4, 7, 11], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": true, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/common/samples/WebApplication1/src/WebApplication1/Startup.cs"} -->

````c#

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

Adding services to the services container makes them available within your application via [dependency injection](dependency-injection.md).

The `ConfigureServices` method is also where you should add configuration option classes that you would like to have available in your application. See the [Configuration](configuration.md) topic to learn more about configuring options.

  ## Services Available in Startup

ASP.NET Core provides certain application services and objects during your application's startup. You can request certain sets of these services by simply including the appropriate interface as a parameter on your `Startup` class's constructor or one of its `Configure` or `ConfigureServices` methods. The services available to each method in the `Startup` class are described below. The framework services and objects include:

IApplicationBuilder
   Used to build the application request pipeline. Available only to the `Configure` method in `Startup`. Learn more about [Request Features](request-features.md).

IHostingEnvironment
   Provides the current `EnvironmentName`, `ContentRootPath`, `WebRootPath`, and web root file provider. Available to the `Startup` constructor and `Configure` method.

ILoggerFactory
   Provides a mechanism for creating loggers. Available to the `Startup` constructor and `Configure` method. Learn more about [Logging](logging.md).

IServiceCollection
   The current set of services configured in the container. Available only to the `ConfigureServices` method, and used by that method to configure the services available to an application.

Looking at each method in the `Startup` class in the order in which they are called, the following services may be requested as parameters:

Startup Constructor - `IHostingEnvironment` - `ILoggerFactory`

ConfigureServices - `IServiceCollection`

Configure - `IApplicationBuilder` - `IHostingEnvironment` - `ILoggerFactory`

Note: Although `ILoggerFactory` is available in the constructor, it is typically configured in the `Configure` method. Learn more about [Logging](logging.md).

  ## Additional Resources

* [Working with Multiple Environments](environments.md)

* [Middleware](middleware.md)

* [Open Web Interface for .NET (OWIN)](owin.md)
