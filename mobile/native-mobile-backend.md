---
uid: mobile/native-mobile-backend
---
  # Creating Backend Services for Native Mobile Applications

By [Steve Smith](http://ardalis.com)

Mobile apps can easily communicate with ASP.NET Core backend services.

[View or download sample backend services code](https://github.com/aspnet/Docs/tree/master/aspnet/mobile/native-mobile-backend/sample)

  ## The Sample Native Mobile App

This tutorial demonstrates how to create backend services using ASP.NET Core MVC to support native mobile apps. It uses the [Xamarin Forms ToDoRest app](https://developer.xamarin.com/guides/xamarin-forms/web-services/consuming/rest/) as its native client, which includes separate native clients for Android, iOS, Windows Universal, and Window Phone devices. You can follow the linked tutorial to create the native app (and install the necessary free Xamarin tools), as well as download the Xamarin sample solution. The Xamarin sample includes an ASP.NET Web API 2 services project, which this article's ASP.NET Core app replaces (with no changes required by the client).

![image](native-mobile-backend/_static/todo-android.png)

  ### Features

The ToDoRest app supports listing, adding, deleting, and updating To-Do items. Each item has an ID, a Name, Notes, and a property indicating whether it's been Done yet.

The main view of the items, as shown above, lists each item's name and indicates if it is done with a checkmark.

Tapping the `+` icon opens an add item dialog:

![image](native-mobile-backend/_static/todo-android-new-item.png)

Tapping an item on the main list screen opens up an edit dialog where the item's Name, Notes, and Done settings can be modified, or the item can be deleted:

![image](native-mobile-backend/_static/todo-android-edit-item.png)

This sample is configured by default to use backend services hosted at developer.xamarin.com, which allow read-only operations. To test it out yourself against the ASP.NET Core app created in the next section running on your computer, you'll need to update the app's `RestUrl` constant. Navigate to the `ToDoREST` project and open the *Constants.cs* file. Replace the `RestUrl` with a URL that includes your machine's IP address (not localhost or 127.0.0.1, since this address is used from the device emulator, not from your machine). Include the port number as well (5000). In order to test that your services work with a device, ensure you don't have an active firewall blocking access to this port.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "csharp"} -->

````csharp

   // URL of REST service (Xamarin ReadOnly Service)
   //public static string RestUrl = "http://developer.xamarin.com:8081/api/todoitems{0}";

   // use your machine's IP address
   public static string RestUrl = "http://192.168.1.207:5000/api/todoitems/{0}";
   ````

  ## Creating the ASP.NET Core Project

Create a new ASP.NET Core Web Application in Visual Studio. Choose the Web API template and No Authentication. Name the project *ToDoApi*.

![image](native-mobile-backend/_static/web-api-template.png)

The application should respond to all requests made to port 5000. Update *Program.cs* to include `.UseUrls("http://*:5000")` to achieve this:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [3], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mobile/native-mobile-backend/sample/ToDoApi/src/ToDoApi/Program.cs"} -->

````c#

   var host = new WebHostBuilder()
       .UseKestrel()
       .UseUrls("http://*:5000")
       .UseContentRoot(Directory.GetCurrentDirectory())
       .UseIISIntegration()
       .UseStartup<Startup>()
       .Build();

   ````

Note: Make sure you run the application directly, rather than behind IIS Express, which ignores non-local requests by default. Run `dotnet run` from the command line, or choose the application name profile from the Debug Target dropdown in the Visual Studio toolbar.

Add a model class to represent To-Do items. Mark required fields using the `[Required]` attribute:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mobile/native-mobile-backend/sample/ToDoApi/src/ToDoApi/Models/ToDoItem.cs"} -->

````c#

   using System.ComponentModel.DataAnnotations;

   namespace ToDoApi.Models
   {
       public class ToDoItem
       {
           [Required]
           public string ID { get; set; }

           [Required]
           public string Name { get; set; }

           [Required]
           public string Notes { get; set; }

           public bool Done { get; set; }
       }
   }
   ````

The API methods require some way to work with data. Use the same `IToDoRepository` interface the original Xamarin sample uses:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mobile/native-mobile-backend/sample/ToDoApi/src/ToDoApi/Interfaces/IToDoRepository.cs"} -->

````c#

   using System.Collections.Generic;
   using ToDoApi.Models;

   namespace ToDoApi.Interfaces
   {
       public interface IToDoRepository
       {
           bool DoesItemExist(string id);
           IEnumerable<ToDoItem> All { get; }
           ToDoItem Find(string id);
           void Insert(ToDoItem item);
           void Update(ToDoItem item);
           void Delete(string id);
       }
   }
   ````

For this sample, the implementation just uses a private collection of items:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mobile/native-mobile-backend/sample/ToDoApi/src/ToDoApi/Services/ToDoRepository.cs"} -->

````c#

   using System.Collections.Generic;
   using System.Linq;
   using ToDoApi.Interfaces;
   using ToDoApi.Models;

   namespace ToDoApi.Services
   {
       public class ToDoRepository : IToDoRepository
       {
           private List<ToDoItem> _toDoList;

           public ToDoRepository()
           {
               InitializeData();
           }

           public IEnumerable<ToDoItem> All
           {
               get { return _toDoList; }
           }

           public bool DoesItemExist(string id)
           {
               return _toDoList.Any(item => item.ID == id);
           }

           public ToDoItem Find(string id)
           {
               return _toDoList.FirstOrDefault(item => item.ID == id);
           }

           public void Insert(ToDoItem item)
           {
               _toDoList.Add(item);
           }

           public void Update(ToDoItem item)
           {
               var todoItem = this.Find(item.ID);
               var index = _toDoList.IndexOf(todoItem);
               _toDoList.RemoveAt(index);
               _toDoList.Insert(index, item);
           }

           public void Delete(string id)
           {
               _toDoList.Remove(this.Find(id));
           }

           private void InitializeData()
           {
               _toDoList = new List<ToDoItem>();

               var todoItem1 = new ToDoItem
               {
                   ID = "6bb8a868-dba1-4f1a-93b7-24ebce87e243",
                   Name = "Learn app development",
                   Notes = "Attend Xamarin University",
                   Done = true
               };

               var todoItem2 = new ToDoItem
               {
                   ID = "b94afb54-a1cb-4313-8af3-b7511551b33b",
                   Name = "Develop apps",
                   Notes = "Use Xamarin Studio/Visual Studio",
                   Done = false
               };

               var todoItem3 = new ToDoItem
               {
                   ID = "ecfa6f80-3671-4911-aabe-63cc442c1ecf",
                   Name = "Publish apps",
                   Notes = "All app stores",
                   Done = false,
               };

               _toDoList.Add(todoItem1);
               _toDoList.Add(todoItem2);
               _toDoList.Add(todoItem3);
           }
       }
   }
   ````

Configure the implementation in *Startup.cs*:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [6], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mobile/native-mobile-backend/sample/ToDoApi/src/ToDoApi/Startup.cs"} -->

````c#

   public void ConfigureServices(IServiceCollection services)
   {
       // Add framework services.
       services.AddMvc();

       services.AddSingleton<IToDoRepository,ToDoRepository>();
   }


   ````

At this point, you're ready to create the *ToDoItemsController*.

Tip: Learn more about creating web APIs in [Building Your First Web API with ASP.NET Core MVC and Visual Studio](../tutorials/first-web-api.md).

  ## Creating the Controller

Add a new controller to the project, *ToDoItemsController*. It should inherit from Microsoft.AspNetCore.Mvc.Controller. Add a `Route` attribute to indicate that the controller will handle requests made to paths starting with `api/todoitems`. The `[controller]` token in the route is replaced by the name of the controller (omitting the `Controller` suffix), and is especially helpful for global routes. Learn more about [routing](../fundamentals/routing.md).

The controller requires an `IToDoRepository` to function; request an instance of this type through the controller's constructor. At runtime, this instance will be provided using the framework's support for [dependency injection](../fundamentals/dependency-injection.md).

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [9, 14], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mobile/native-mobile-backend/sample/ToDoApi/src/ToDoApi/Controllers/ToDoItemsController.cs"} -->

````c#

   using System;
   using Microsoft.AspNetCore.Http;
   using Microsoft.AspNetCore.Mvc;
   using ToDoApi.Interfaces;
   using ToDoApi.Models;

   namespace ToDoApi.Controllers
   {
       [Route("api/[controller]")]
       public class ToDoItemsController : Controller
       {
           private readonly IToDoRepository _toDoRepository;

           public ToDoItemsController(IToDoRepository toDoRepository)
           {
               _toDoRepository = toDoRepository;
           }

   ````

This API supports four different HTTP verbs to perform CRUD (Create, Read, Update, Delete) operations on the data source. The simplest of these is the Read operation, which corresponds to an HTTP GET request.

  ### Reading Items

Requesting a list of items is done with a GET request to the `List` method. The `[HttpGet]` attribute on the `List` method indicates that this action should only handle GET requests. The route for this action is the route specified on the controller. You don't necessarily need to use the action name as part of the route. You just need to ensure each action has a unique and unambiguous route. Routing attributes can be applied at both the controller and method levels to build up specific routes.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mobile/native-mobile-backend/sample/ToDoApi/src/ToDoApi/Controllers/ToDoItemsController.cs"} -->

````c#

   [HttpGet]
   public IActionResult List()
   {
       return Ok(_toDoRepository.All);
   }

   ````

The `List` method returns a 200 OK response code and all of the ToDo items, serialized as JSON.

You can test your new API method using a variety of tools, such as [Postman](https://www.getpostman.com/docs/), shown here:

![image](native-mobile-backend/_static/postman-get.png)

  ### Creating Items

By convention, creating new data items is mapped to the HTTP POST verb. The `Create` method has an `[HttpPost]` attribute applied to it, and accepts an ID parameter and a `ToDoItem` instance. The HTTP verb attributes, like `[HttpPost]`, optionally accept a route template string (`{id}` in this example). This has the same effect as adding a `[Route]` attribute to the action. Since the `item` argument will be passed in the body of the POST, this parameter is decorated with the `[FromBody]` attribute.

Inside the method, the item is checked for validity and prior existence in the data store, and if no issues occur, it is added using the repository. Checking `ModelState.IsValid` performs [model validation](../mvc/models/validation.md), and should be done in every API method that accepts user input.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mobile/native-mobile-backend/sample/ToDoApi/src/ToDoApi/Controllers/ToDoItemsController.cs"} -->

````c#

   [HttpPost("{id}")]
   public IActionResult Create(string id, [FromBody]ToDoItem item)
   {
       try
       {
           if (item == null || !ModelState.IsValid)
           {
               return BadRequest(ErrorCode.TodoItemNameAndNotesRequired.ToString());
           }
           bool itemExists = _toDoRepository.DoesItemExist(item.ID);
           if (itemExists)
           {
               return StatusCode(StatusCodes.Status409Conflict, ErrorCode.TodoItemIDInUse.ToString());
           }
           _toDoRepository.Insert(item);
       }
       catch (Exception)
       {
           return BadRequest(ErrorCode.CouldNotCreateItem.ToString());
       }
       return Ok(item);
   }

   ````

The sample uses an enum containing error codes that are passed to the mobile client:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mobile/native-mobile-backend/sample/ToDoApi/src/ToDoApi/Controllers/ToDoItemsController.cs"} -->

````c#

   public enum ErrorCode
   {
       TodoItemNameAndNotesRequired,
       TodoItemIDInUse,
       RecordNotFound,
       CouldNotCreateItem,
       CouldNotUpdateItem,
       CouldNotDeleteItem
   }

   ````

Test adding new items using Postman by choosing the POST verb providing the new object in JSON format in the Body of the request. You should also add a request header specifying a `Content-Type` of `application/json`.

![image](native-mobile-backend/_static/postman-post.png)

The method returns the newly created item in the response.

  ### Updating Items

Modifying records is done using HTTP PUT requests. Other than this change, the `Edit` method is almost identical to `Create`. Note that if the record isn't found, the `Edit` action will return a `NotFound` (404) response.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mobile/native-mobile-backend/sample/ToDoApi/src/ToDoApi/Controllers/ToDoItemsController.cs"} -->

````c#

   [HttpPut("{id}")]
   public IActionResult Edit(string id, [FromBody] ToDoItem item)
   {
       try
       {
           if (item == null || !ModelState.IsValid)
           {
               return BadRequest(ErrorCode.TodoItemNameAndNotesRequired.ToString());
           }
           var existingItem = _toDoRepository.Find(id);
           if (existingItem == null)
           {
               return NotFound(ErrorCode.RecordNotFound.ToString());
           }
           _toDoRepository.Update(item);
       }
       catch (Exception)
       {
           return BadRequest(ErrorCode.CouldNotUpdateItem.ToString());
       }
       return NoContent();
   }

   ````

To test with Postman, change the verb to PUT and add the ID of the record being updated to the URL. Specify the updated object data in the Body of the request.

![image](native-mobile-backend/_static/postman-put.png)

This method returns a `NoContent` (204) response when successful, for consistency with the pre-existing API.

  ### Deleting Items

Deleting records is accomplished by making DELETE requests to the service, and passing the ID of the item to be deleted. As with updates, requests for items that don't exist will receive `NotFound` responses. Otherwise, a successful request will get a `NoContent` (204) response.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mobile/native-mobile-backend/sample/ToDoApi/src/ToDoApi/Controllers/ToDoItemsController.cs"} -->

````c#

   [HttpDelete("{id}")]
   public IActionResult Delete(string id)
   {
       try
       {
           var item = _toDoRepository.Find(id);
           if (item == null)
           {
               return NotFound(ErrorCode.RecordNotFound.ToString());
           }
           _toDoRepository.Delete(id);
       }
       catch (Exception)
       {
           return BadRequest(ErrorCode.CouldNotDeleteItem.ToString());
       }
       return NoContent();
   }

   ````

Note that when testing the delete functionality, nothing is required in the Body of the request.

![image](native-mobile-backend/_static/postman-delete.png)

  ## Common Web API Conventions

As you develop the backend services for your app, you will want to come up with a consistent set of conventions or policies for handling cross-cutting concerns. For example, in the service shown above, requests for specific records that weren't found received a `NotFound` response, rather than a `BadRequest` response. Similarly, commands made to this service that passed in model bound types always checked `ModelState.IsValid` and returned a `BadRequest` for invalid model types.

Once you've identified a common policy for your APIs, you can usually encapsulate it in a [filter](../mvc/controllers/filters.md). Learn more about [how to encapsulate common API policies in ASP.NET Core MVC applications](https://msdn.microsoft.com/en-us/magazine/mt767699.aspx).
