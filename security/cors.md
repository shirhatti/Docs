---
uid: security/cors
---
  # Enabling Cross-Origin Requests (CORS)

By [Mike Wasson](https://github.com/mikewasson) and [Shayne Boyer](https://twitter.com/spboyer)

Browser security prevents a web page from making AJAX requests to another domain. This restriction is called the *same-origin policy*, and prevents a malicious site from reading sensitive data from another site. However, sometimes you might want to let other sites make cross-origin requests to your web app.

[Cross Origin Resource Sharing](http://www.w3.org/TR/cors/) (CORS) is a W3C standard that allows a server to relax the same-origin policy. Using CORS, a server can explicitly allow some cross-origin requests while rejecting others. CORS is safer and more flexible than earlier techniques such as [JSONP](http://en.wikipedia.org/wiki/JSONP). This topic shows how to enable CORS in your ASP.NET Core application.

  ## What is "same origin"?

Two URLs have the same origin if they have identical schemes, hosts, and ports. ([RFC 6454](http://tools.ietf.org/html/rfc6454))

These two URLs have the same origin:

* http://example.com/foo.html

* http://example.com/bar.html

These URLs have different origins than the previous two:

* http://example.net - Different domain

* http://example.com:9000/foo.html - Different port

* https://example.com/foo.html - Different scheme

* http://www.example.com/foo.html - Different subdomain

Note: Internet Explorer does not consider the port when comparing origins.

  ## Setting up CORS

To setup CORS for your application add the `Microsoft.AspNetCore.Cors` package to your project.

Add the CORS services in Startup.cs:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "csharp", "source": "/Users/shirhatti/src/Docs/aspnet/security/cors/sample/src/CorsExamples/CorsExample1/Startup.cs"} -->

````csharp

   public void ConfigureServices(IServiceCollection services)
   {
       services.AddCors();
   }

   ````

  ## Enabling CORS with middleware

To enable CORS for your entire application add the CORS middleware to your request pipeline using the `UseCors` extension method. Note that the CORS middleware must precede any defined endpoints in your app that you want to support cross-origin requests (ex. before any call to `UseMvc`).

You can specify a cross-origin policy when adding the CORS middleware using the `CorsPolicyBuilder` class. There are two ways to do this. The first is to call UseCors with a lambda:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [11, 12], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "csharp", "source": "/Users/shirhatti/src/Docs/aspnet/security/cors/sample/src/CorsExamples/CorsExample1/Startup.cs"} -->

````csharp

   public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
   {
       loggerFactory.AddConsole();

       if (env.IsDevelopment())
       {
           app.UseDeveloperExceptionPage();
       }

       // Shows UseCors with CorsPolicyBuilder.
       app.UseCors(builder =>
          builder.WithOrigins("http://example.com"));

       app.Run(async (context) =>
       {
           await context.Response.WriteAsync("Hello World!");
       });
   }

   ````

The lambda takes a CorsPolicyBuilder object. I’ll describe all of the configuration options later in this topic. In this example, the policy allows cross-origin requests from "http://example.com" and no other origins.

Note that CorsPolicyBuilder has a fluent API, so you can chain method calls:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [3], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "csharp", "source": "/Users/shirhatti/src/Docs/aspnet/security/cors/sample/src/CorsExamples/CorsExample3/Startup.cs"} -->

````csharp

       app.UseCors(builder =>
           builder.WithOrigins("http://example.com")
                  .AllowAnyHeader()
           );
       

   ````

The second approach is to define one or more named CORS policies, and then select the policy by name at run time.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "csharp", "source": "/Users/shirhatti/src/Docs/aspnet/security/cors/sample/src/CorsExamples/CorsExample2/Startup.cs"} -->

````csharp

   public void ConfigureServices(IServiceCollection services)
   {
       services.AddCors(options =>
       {
           options.AddPolicy("AllowSpecificOrigin",
               builder => builder.WithOrigins("http://example.com"));
       });
   }

   public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
   {
       loggerFactory.AddConsole();

       if (env.IsDevelopment())
       {
           app.UseDeveloperExceptionPage();
       }

       // Shows UseCors with named policy.
       app.UseCors("AllowSpecificOrigin");
       app.Run(async (context) =>
       {
           await context.Response.WriteAsync("Hello World!");
       });
   }

   ````

This example adds a CORS policy named "AllowSpecificOrigin". To select the policy, pass the name to UseCors.

<a name=cors-policy-options></a>

  ## Enabling CORS in MVC

You can alternatively use MVC to apply specific CORS per action, per controller, or globally for all controllers. When using MVC to enable CORS the same CORS services are used, but the CORS middleware is not.

  ### Per action

To specify a CORS policy for a specific action add the `[EnableCors]` attribute to the action. Specify the policy name.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "csharp", "source": "/Users/shirhatti/src/Docs/aspnet/security/cors/sample/src/CorsExamples/CorsMVC/Controllers/HomeController.cs"} -->

````csharp


   [EnableCors("AllowSpecificOrigin")]
   public class HomeController : Controller
   {
       [EnableCors("AllowSpecificOrigin")]
       public IActionResult Index()
       {
           return View();
       }

   ````

  ### Per controller

To specify the CORS policy for a specific controller add the `[EnableCors]` attribute to the controller class. Specify the policy name.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "csharp", "source": "/Users/shirhatti/src/Docs/aspnet/security/cors/sample/src/CorsExamples/CorsMVC/Controllers/HomeController.cs"} -->

````csharp

   [EnableCors("AllowSpecificOrigin")]
   public class HomeController : Controller
   {

   ````

  ### Globally

You can enable CORS globally for all controllers by adding the `CorsAuthorizationFilterFactory` filter to the global filter collection:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "csharp", "source": "/Users/shirhatti/src/Docs/aspnet/security/cors/sample/src/CorsExamples/CorsMVC/Startup2.cs"} -->

````csharp

   public void ConfigureServices(IServiceCollection services)
   {
       services.AddMvc();
       services.Configure<MvcOptions>(options =>
       {
           options.Filters.Add(new CorsAuthorizationFilterFactory("AllowSpecificOrigin"));
       });
   }

   ````

The precedence order is: Action, controller, global. Action-level policies take precedence over controller-level policies, and controller-level policies take precedence over global policies.

  ### Disable CORS

To disable CORS for a controller or action, use the `[DisableCors]` attribute.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "csharp", "source": "/Users/shirhatti/src/Docs/aspnet/security/cors/sample/src/CorsExamples/CorsMVC/Controllers/HomeController.cs"} -->

````csharp

       [DisableCors]
       public IActionResult About()
       {
           return View();
       }

   ````

  ## CORS policy options

This section describes the various options that you can set in a CORS policy.

* [Set the allowed origins](#set-the-allowed-origins)

* [Set the allowed HTTP methods](#set-the-allowed-http-methods)

* [Set the allowed request headers](#set-the-allowed-request-headers)

* [Set the exposed response headers](#set-the-exposed-response-headers)

* [Credentials in cross-origin requests](#credentials-in-cross-origin-requests)

* [Set the preflight expiration time](#set-the-preflight-expiration-time)

For some options it may be helpful to read [How CORS works](#how-cors-works) first.

  ### Set the allowed origins

To allow one or more specific origins:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "csharp", "source": "/Users/shirhatti/src/Docs/aspnet/security/cors/sample/src/CorsExamples/CorsExample4/Startup.cs"} -->

````csharp

   options.AddPolicy("AllowSpecificOrigins",
   builder =>
   {
       builder.WithOrigins("http://example.com", "http://www.contoso.com");
   });

   ````

To allow all origins:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "csharp", "source": "/Users/shirhatti/src/Docs/aspnet/security/cors/sample/src/CorsExamples/CorsExample4/Startup.cs"} -->

````csharp

   options.AddPolicy("AllowAllOrigins",
       builder =>
       {
           builder.AllowAnyOrigin();
       });

   ````

Consider carefully before allowing requests from any origin. It means that literally any website can make AJAX calls to your app.

  ### Set the allowed HTTP methods

To specify which HTTP methods are allowed to access the resource.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "csharp", "source": "/Users/shirhatti/src/Docs/aspnet/security/cors/sample/src/CorsExamples/CorsExample4/Startup.cs"} -->

````csharp

   options.AddPolicy("AllowSpecificMethods",
       builder =>
       {
           builder.WithOrigins("http://example.com")
                  .WithMethods("GET", "POST", "HEAD");
       });

   ````

To allow all HTTP methods:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "csharp", "source": "/Users/shirhatti/src/Docs/aspnet/security/cors/sample/src/CorsExamples/CorsExample4/Startup.cs"} -->

````csharp

   options.AddPolicy("AllowAllMethods",
       builder =>
       {
           builder.WithOrigins("http://example.com")
                  .AllowAnyMethod();
       });

   ````

This affects pre-flight requests and Access-Control-Allow-Methods header.

  ### Set the allowed request headers

A CORS preflight request might include an Access-Control-Request-Headers header, listing the HTTP headers set by the application (the so-called "author request headers").

To whitelist specific headers:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "csharp", "source": "/Users/shirhatti/src/Docs/aspnet/security/cors/sample/src/CorsExamples/CorsExample4/Startup.cs"} -->

````csharp

   options.AddPolicy("AllowHeaders",
       builder =>
       {
           builder.WithOrigins("http://example.com")
                  .WithHeaders("accept", "content-type", "origin", "x-custom-header");
       });

   ````

To allow all author request headers:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "csharp", "source": "/Users/shirhatti/src/Docs/aspnet/security/cors/sample/src/CorsExamples/CorsExample4/Startup.cs"} -->

````csharp

   options.AddPolicy("AllowAllHeaders",
       builder =>
       {
           builder.WithOrigins("http://example.com")
                  .AllowAnyHeader();
       });

   ````

Browsers are not entirely consistent in how they set Access-Control-Request-Headers. If you set headers to anything other than "*", you should include at least "accept", "content-type", and "origin", plus any custom headers that you want to support.

  ### Set the exposed response headers

By default, the browser does not expose all of the response headers to the application. (See [http://www.w3.org/TR/cors/#simple-response-header](http://www.w3.org/TR/cors/#simple-response-header).) The response headers that are available by default are:

* Cache-Control

* Content-Language

* Content-Type

* Expires

* Last-Modified

* Pragma

The CORS spec calls these *simple response headers*. To make other headers available to the application:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "csharp", "source": "/Users/shirhatti/src/Docs/aspnet/security/cors/sample/src/CorsExamples/CorsExample4/Startup.cs"} -->

````csharp

   options.AddPolicy("ExposeResponseHeaders",
       builder =>
       {
           builder.WithOrigins("http://example.com")
                  .WithExposedHeaders("x-custom-header");
       });

   ````

  ### Credentials in cross-origin requests

Credentials require special handling in a CORS request. By default, the browser does not send any credentials with a cross-origin request. Credentials include cookies as well as HTTP authentication schemes. To send credentials with a cross-origin request, the client must set XMLHttpRequest.withCredentials to true.

Using XMLHttpRequest directly:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "js"} -->

````js

   var xhr = new XMLHttpRequest();
   xhr.open('get', 'http://www.example.com/api/test');
   xhr.withCredentials = true;
   ````

In jQuery:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "js"} -->

````js

   $.ajax({
       type: 'get',
       url: 'http://www.example.com/home',
       xhrFields: {
           withCredentials: true
       }
   ````

In addition, the server must allow the credentials. To allow cross-origin credentials:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "csharp", "source": "/Users/shirhatti/src/Docs/aspnet/security/cors/sample/src/CorsExamples/CorsExample4/Startup.cs"} -->

````csharp

   options.AddPolicy("AllowCredentials",
       builder =>
       {
           builder.WithOrigins("http://example.com")
                  .AllowCredentials();
       });

   ````

Now the HTTP response will include an Access-Control-Allow-Credentials header, which tells the browser that the server allows credentials for a cross-origin request.

If the browser sends credentials, but the response does not include a valid Access-Control-Allow-Credentials header, the browser will not expose the response to the application, and the AJAX request fails.

Be very careful about allowing cross-origin credentials, because it means a website at another domain can send a logged-in user’s credentials to your app on the user’s behalf, without the user being aware. The CORS spec also states that setting origins to "*" (all origins) is invalid if the Access-Control-Allow-Credentials header is present.

  ### Set the preflight expiration time

The Access-Control-Max-Age header specifies how long the response to the preflight request can be cached. To set this header:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "csharp", "source": "/Users/shirhatti/src/Docs/aspnet/security/cors/sample/src/CorsExamples/CorsExample4/Startup.cs"} -->

````csharp

   options.AddPolicy("SetPreflightExpiration",
       builder =>
       {
           builder.WithOrigins("http://example.com")
                  .SetPreflightMaxAge(TimeSpan.FromSeconds(2520));
       });

   ````

<a name=cors-how-cors-works></a>

  ## How CORS works

This section describes what happens in a CORS request, at the level of the HTTP messages. It’s important to understand how CORS works, so that you can configure the your CORS policy correctly, and troubleshoot if things don’t work as you expect.

The CORS specification introduces several new HTTP headers that enable cross-origin requests. If a browser supports CORS, it sets these headers automatically for cross-origin requests; you don’t need to do anything special in your JavaScript code.

Here is an example of a cross-origin request. The "Origin" header gives the domain of the site that is making the request:

<!-- literal_block {"ids": [], "names": [], "backrefs": [], "dupnames": [], "xml:space": "preserve", "classes": []} -->

````

   GET http://myservice.azurewebsites.net/api/test HTTP/1.1
   Referer: http://myclient.azurewebsites.net/
   Accept: */*
   Accept-Language: en-US
   Origin: http://myclient.azurewebsites.net
   Accept-Encoding: gzip, deflate
   User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.2; WOW64; Trident/6.0)
   Host: myservice.azurewebsites.net
   ````

If the server allows the request, it sets the Access-Control-Allow-Origin header. The value of this header either matches the Origin header, or is the wildcard value "*", meaning that any origin is allowed.:

<!-- literal_block {"ids": [], "names": [], "backrefs": [], "dupnames": [], "xml:space": "preserve", "classes": []} -->

````

   HTTP/1.1 200 OK
   Cache-Control: no-cache
   Pragma: no-cache
   Content-Type: text/plain; charset=utf-8
   Access-Control-Allow-Origin: http://myclient.azurewebsites.net
   Date: Wed, 20 May 2015 06:27:30 GMT
   Content-Length: 12

   Test message
   ````

If the response does not include the Access-Control-Allow-Origin header, the AJAX request fails. Specifically, the browser disallows the request. Even if the server returns a successful response, the browser does not make the response available to the client application.

  ### Preflight Requests

For some CORS requests, the browser sends an additional request, called a "preflight request", before it sends the actual request for the resource. The browser can skip the preflight request if the following conditions are true:

* The request method is GET, HEAD, or POST, and

* The application does not set any request headers other than Accept, Accept-Language, Content-Language, Content-Type, or Last-Event-ID, and

* The Content-Type header (if set) is one of the following:

  * application/x-www-form-urlencoded

  * multipart/form-data

  * text/plain

The rule about request headers applies to headers that the application sets by calling setRequestHeader on the XMLHttpRequest object. (The CORS specification calls these "author request headers".) The rule does not apply to headers the browser can set, such as User-Agent, Host, or Content-Length.

Here is an example of a preflight request:

<!-- literal_block {"ids": [], "names": [], "backrefs": [], "dupnames": [], "xml:space": "preserve", "classes": []} -->

````

   OPTIONS http://myservice.azurewebsites.net/api/test HTTP/1.1
   Accept: */*
   Origin: http://myclient.azurewebsites.net
   Access-Control-Request-Method: PUT
   Access-Control-Request-Headers: accept, x-my-custom-header
   Accept-Encoding: gzip, deflate
   User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.2; WOW64; Trident/6.0)
   Host: myservice.azurewebsites.net
   Content-Length: 0
   ````

The pre-flight request uses the HTTP OPTIONS method. It includes two special headers:

* Access-Control-Request-Method: The HTTP method that will be used for the actual request.

* Access-Control-Request-Headers: A list of request headers that the application set on the actual request. (Again, this does not include headers that the browser sets.)

Here is an example response, assuming that the server allows the request:

<!-- literal_block {"ids": [], "names": [], "backrefs": [], "dupnames": [], "xml:space": "preserve", "classes": []} -->

````

   HTTP/1.1 200 OK
   Cache-Control: no-cache
   Pragma: no-cache
   Content-Length: 0
   Access-Control-Allow-Origin: http://myclient.azurewebsites.net
   Access-Control-Allow-Headers: x-my-custom-header
   Access-Control-Allow-Methods: PUT
   Date: Wed, 20 May 2015 06:33:22 GMT
   ````

The response includes an Access-Control-Allow-Methods header that lists the allowed methods, and optionally an Access-Control-Allow-Headers header, which lists the allowed headers. If the preflight request succeeds, the browser sends the actual request, as described earlier.
