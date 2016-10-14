---
uid: fundamentals/owin
---
  # Open Web Interface for .NET (OWIN)

By [Steve Smith](http://ardalis.com) and  [Rick Anderson](https://twitter.com/RickAndMSFT)

ASP.NET Core supports the Open Web Interface for .NET (OWIN). OWIN allows web apps to be decoupled from web servers. It defines a standard way for middleware to be used in a pipeline to handle requests and associated responses. ASP.NET Core applications and middleware can interoperate with OWIN-based applications, servers, and middleware.

[View or download sample code](https://github.com/aspnet/Docs/tree/master/aspnet/fundamentals/owin/sample)

  ## Running OWIN middleware in the ASP.NET pipeline

ASP.NET Core's OWIN support is deployed as part of the `Microsoft.AspNetCore.Owin` package. You can import OWIN support into your project by adding this package as a dependency in your *project.json* file:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [4], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "javascript", "source": "/Users/shirhatti/src/Docs/aspnet/fundamentals/owin/sample/src/OwinSample/project.json"} -->

````javascript

     "dependencies": {
       "Microsoft.AspNetCore.Server.IISIntegration": "1.0.0",
       "Microsoft.AspNetCore.Server.Kestrel": "1.0.0",
       "Microsoft.AspNetCore.Owin": "1.0.0"
     },

   ````

OWIN middleware conforms to the [OWIN specification](http://owin.org/spec/spec/owin-1.0.0.html), which requires a `Func<IDictionary<string, object>, Task>` interface, and specific keys be set (such as `owin.ResponseBody`). The following simple OWIN middleware displays "Hello World":

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/fundamentals/owin/sample/src/OwinSample/Startup.cs"} -->

````c#

   public Task OwinHello(IDictionary<string, object> environment)
   {
       string responseText = "Hello World via OWIN";
       byte[] responseBytes = Encoding.UTF8.GetBytes(responseText);

       // OWIN Environment Keys: http://owin.org/spec/spec/owin-1.0.0.html
       var responseStream = (Stream)environment["owin.ResponseBody"];
       var responseHeaders = (IDictionary<string, string[]>)environment["owin.ResponseHeaders"];

       responseHeaders["Content-Length"] = new string[] { responseBytes.Length.ToString(CultureInfo.InvariantCulture) };
       responseHeaders["Content-Type"] = new string[] { "text/plain" };

       return responseStream.WriteAsync(responseBytes, 0, responseBytes.Length);
   }


   ````

The sample signature returns a `Task` and accepts an `IDictionary<string, object>` as required by OWIN.

The following code shows how to add the `OwinHello` middleware (shown above) to the ASP.NET pipeline with the [UseOwin](http://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Builder/OwinExtensions/index.html.md#Microsoft.AspNetCore.Builder.OwinExtensions.UseOwin.md) extension method.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/fundamentals/owin/sample/src/OwinSample/Startup.cs"} -->

````c#

   public void Configure(IApplicationBuilder app)
   {
       app.UseOwin(pipeline =>
       {
           pipeline(next => OwinHello);
       });
   }


   ````

You can configure other actions to take place within the OWIN pipeline.

Note: Response headers should only be modified prior to the first write to the response stream.

Note: Multiple calls to `UseOwin` is discouraged for performance reasons. OWIN components will operate best if grouped together.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#"} -->

````c#

   app.UseOwin(pipeline =>
   {
       pipeline(next =>
       {
           // do something before
           return OwinHello;
           // do something after
       });
   });
   ````

<a name=hosting-on-owin></a>

  ## Using ASP.NET Hosting on an OWIN-based server

OWIN-based servers can host ASP.NET applications. One such server is [Nowin](https://github.com/Bobris/Nowin), a .NET OWIN web server. In the sample for this article, I've included a project that references Nowin and uses it to create an `IServer` capable of self-hosting ASP.NET Core.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [15], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/fundamentals/owin/sample/src/NowinSample/NowinServer.cs"} -->

````c#

   using System;
   using System.Collections.Generic;
   using System.Linq;
   using System.Net;
   using System.Threading.Tasks;
   using Microsoft.AspNetCore.Hosting.Server;
   using Microsoft.AspNetCore.Hosting.Server.Features;
   using Microsoft.AspNetCore.Http.Features;
   using Microsoft.AspNetCore.Owin;
   using Microsoft.Extensions.Options;
   using Nowin;

   namespace NowinSample
   {
       public class NowinServer : IServer
       {
           private INowinServer _nowinServer;
           private ServerBuilder _builder;

           public IFeatureCollection Features { get; } = new FeatureCollection();

           public NowinServer(IOptions<ServerBuilder> options)
           {
               Features.Set<IServerAddressesFeature>(new ServerAddressesFeature());
               _builder = options.Value;
           }

           public void Start<TContext>(IHttpApplication<TContext> application)
           {
               // Note that this example does not take into account of Nowin's "server.OnSendingHeaders" callback.
               // Ideally we should ensure this method is fired before disposing the context. 
               Func<IDictionary<string, object>, Task> appFunc = async env =>
               {
                   // The reason for 2 level of wrapping is because the OwinFeatureCollection isn't mutable
                   // so features can't be added
                   var features = new FeatureCollection(new OwinFeatureCollection(env));

                   var context = application.CreateContext(features);
                   try
                   {
                       await application.ProcessRequestAsync(context);
                   }
                   catch (Exception ex)
                   {
                       application.DisposeContext(context, ex);
                       throw;
                   }

                   application.DisposeContext(context, null);
               };

               // Add the web socket adapter so we can turn OWIN websockets into ASP.NET Core compatible web sockets.
               // The calling pattern is a bit different
               appFunc = OwinWebSocketAcceptAdapter.AdaptWebSockets(appFunc);

               // Get the server addresses
               var address = Features.Get<IServerAddressesFeature>().Addresses.First();

               var uri = new Uri(address);
               var port = uri.Port;
               IPAddress ip;
               if (!IPAddress.TryParse(uri.Host, out ip))
               {
                   ip = IPAddress.Loopback;
               }

               _nowinServer = _builder.SetAddress(ip)
                                       .SetPort(port)
                                       .SetOwinApp(appFunc)
                                       .Build();
               _nowinServer.Start();
           }

           public void Dispose()
           {
               _nowinServer?.Dispose();
           }
       }
   }
   ````

`IServer` is an interface that requires an `Features` property and a `Start` method.

`Start` is responsible for configuring and starting the server, which in this case is done through a series of fluent API calls that set addresses parsed from the IServerAddressesFeature. Note that the fluent configuration of the `_builder` variable specifies that requests will be handled by the `appFunc` defined earlier in the method. This `Func` is called on each request to process incoming requests.

We'll also add an `IWebHostBuilder` extension to make it easy to add and configure the Nowin server.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [11], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/fundamentals/owin/sample/src/NowinSample/NowinWebHostBuilderExtensions.cs"} -->

````c#

   using System;
   using Microsoft.AspNetCore.Hosting.Server;
   using Microsoft.Extensions.DependencyInjection;
   using Nowin;
   using NowinSample;

   namespace Microsoft.AspNetCore.Hosting
   {
       public static class NowinWebHostBuilderExtensions
       {
           public static IWebHostBuilder UseNowin(this IWebHostBuilder builder)
           {
               return builder.ConfigureServices(services =>
               {
                   services.AddSingleton<IServer, NowinServer>();
               });
           }

           public static IWebHostBuilder UseNowin(this IWebHostBuilder builder, Action<ServerBuilder> configure)
           {
               builder.ConfigureServices(services =>
               {
                   services.Configure(configure);
               });
               return builder.UseNowin();
           }
       }
   }
   ````

With this in place, all that's required to run an ASP.NET application using this custom server to call the extension in *Program.cs*:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [15], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/fundamentals/owin/sample/src/NowinSample/Program.cs"} -->

````c#

   using System;
   using System.Collections.Generic;
   using System.IO;
   using System.Linq;
   using System.Threading.Tasks;
   using Microsoft.AspNetCore.Hosting;

   namespace NowinSample
   {
       public class Program
       {
           public static void Main(string[] args)
           {
               var host = new WebHostBuilder()
                   .UseNowin()
                   .UseContentRoot(Directory.GetCurrentDirectory())
                   .UseIISIntegration()
                   .UseStartup<Startup>()
                   .Build();

               host.Run();
           }
       }
   }

   ````

Learn more about ASP.NET [Servers](servers.md).

  ## Run ASP.NET Core on an OWIN-based server and use its WebSockets support

Another example of how OWIN-based servers' features can be leveraged by ASP.NET Core is access to features like WebSockets. The .NET OWIN web server used in the previous example has support for Web Sockets built in, which can be leveraged by an ASP.NET Core application. The example below shows a simple web app that supports Web Sockets and echoes back everything sent to the server through WebSockets.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [7, 9, 10], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": true, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/fundamentals/owin/sample/src/NowinWebSockets/Startup.cs"} -->

````c#

       public class Startup
       {
           public void Configure(IApplicationBuilder app)
           {
               app.Use(async (context, next) =>
               {
                   if (context.WebSockets.IsWebSocketRequest)
                   {
                       WebSocket webSocket = await context.WebSockets.AcceptWebSocketAsync();
                       await EchoWebSocket(webSocket);
                   }
                   else
                   {
                       await next();
                   }
               });

               app.Run(context =>
               {
                   return context.Response.WriteAsync("Hello World");
               });
           }

           private async Task EchoWebSocket(WebSocket webSocket)
           {
               byte[] buffer = new byte[1024];
               WebSocketReceiveResult received = await webSocket.ReceiveAsync(
                   new ArraySegment<byte>(buffer), CancellationToken.None);

               while (!webSocket.CloseStatus.HasValue)
               {
                   // Echo anything we receive
                   await webSocket.SendAsync(new ArraySegment<byte>(buffer, 0, received.Count), 
                       received.MessageType, received.EndOfMessage, CancellationToken.None);

                   received = await webSocket.ReceiveAsync(new ArraySegment<byte>(buffer), 
                       CancellationToken.None);
               }

               await webSocket.CloseAsync(webSocket.CloseStatus.Value, 
                   webSocket.CloseStatusDescription, CancellationToken.None);
           }
       }
   }
   ````

This [sample](https://github.com/aspnet/Docs/tree/master/aspnet/fundamentals/owin/sample) is configured using the same `NowinServer` as the previous one - the only difference is in how the application is configured in its `Configure` method. A test using [a simple websocket client](https://chrome.google.com/webstore/detail/simple-websocket-client/pfdhoblngboilpfeibdedpjgfnlcodoo?hl=en) demonstrates  the application:

![image](owin/_static/websocket-test.png)

  ## OWIN environment

You can construct a OWIN environment using the `HttpContext`.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#"} -->

````c#

   var environment = new OwinEnvironment(HttpContext);
   var features = new OwinFeatureCollection(environment);
   ````

  ## OWIN keys

OWIN depends on an `IDictionary<string,object>` object to communicate information throughout an HTTP Request/Response exchange. ASP.NET Core implements the keys listed below. See the [primary specification, extensions](http://owin.org/#spec), and [OWIN Key Guidelines and Common Keys](http://owin.org/spec/spec/CommonKeys.html).

  ### Request Data (OWIN v1.0.0)

<!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- table -->

  ### Request Data (OWIN v1.1.0)

<!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- table -->

  ### Response Data (OWIN v1.0.0)

<!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- table -->

  ### Other Data (OWIN v1.0.0)

<!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- table -->

  ### Common Keys

<!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- table -->

  ### SendFiles v0.3.0

<!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- table -->

  ### Opaque v0.3.0

<!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- table -->

  ### WebSocket v0.3.0

<!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- table -->

  ## Additional Resources

* [Middleware](middleware.md)

* [Servers](servers.md)
