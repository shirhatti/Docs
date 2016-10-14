---
uid: mvc/controllers/testing
---
  # Testing Controller Logic

By [Steve Smith](http://ardalis.com)

Controllers in ASP.NET MVC apps should be small and focused on user-interface concerns. Large controllers that deal with non-UI concerns are more difficult to test and maintain.

[View or download sample from GitHub](https://github.com/aspnet/Docs/tree/master/aspnet/mvc/controllers/testing/sample)

  ## Why Test Controllers

Controllers are a central part of any ASP.NET Core MVC application. As such, you should have confidence they behave as intended for your app. Automated tests can provide you with this confidence and can detect errors before they reach production. It's important to avoid placing unnecessary responsibilities within your controllers and ensure your tests focus only on controller responsibilities.

Controller logic should be minimal and not be focused on business logic or infrastructure concerns (for example, data access). Test controller logic, not the framework. Test how the controller *behaves* based on valid or invalid inputs. Test controller responses based on the result of the business operation it performs.

Typical controller responsibilities:

* Verify `ModelState.IsValid`

* Return an error response if `ModelState` is invalid

* Retrieve a business entity from persistence

* Perform an action on the business entity

* Save the business entity to persistence

* Return an appropriate `IActionResult`

  ## Unit Testing

[Unit testing](https://docs.microsoft.com/en-us/dotnet/articles/core/testing/unit-testing-with-dotnet-test) involves testing a part of an app in isolation from its infrastructure and dependencies. When unit testing controller logic, only the contents of a single action is tested, not the behavior of its dependencies or of the framework itself. As you unit test your controller actions, make sure you focus only on its behavior. A controller unit test avoids things like [filters](filters.md), [routing](../../fundamentals/routing.md), or [model binding](../models/model-binding.md). By focusing on testing just one thing, unit tests are generally simple to write and quick to run. A well-written set of unit tests can be run frequently without much overhead. However, unit tests do not detect issues in the interaction between components, which is the purpose of [integration testing](xref:mvc/controllers/testing#integration-testing).

If you've writing custom filters, routes, etc, you should unit test them, but not as part of your tests on a particular controller action. They should be tested in isolation.

Tip: [Create and run unit tests with Visual Studio](https://www.visualstudio.com/en-us/get-started/code/create-and-run-unit-tests-vs).

To demonstrate unit testing, review the following controller. It displays a list of brainstorming sessions and allows new brainstorming sessions to be created with a POST:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [12, 16, 21, 42, 43], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/controllers/testing/sample/TestingControllersSample/src/TestingControllersSample/Controllers/HomeController.cs"} -->

````c#

   using System;
   using System.ComponentModel.DataAnnotations;
   using System.Linq;
   using System.Threading.Tasks;
   using Microsoft.AspNetCore.Mvc;
   using TestingControllersSample.Core.Interfaces;
   using TestingControllersSample.Core.Model;
   using TestingControllersSample.ViewModels;

   namespace TestingControllersSample.Controllers
   {
       public class HomeController : Controller
       {
           private readonly IBrainstormSessionRepository _sessionRepository;

           public HomeController(IBrainstormSessionRepository sessionRepository)
           {
               _sessionRepository = sessionRepository;
           }

           public async Task<IActionResult> Index()
           {
               var sessionList = await _sessionRepository.ListAsync();

               var model = sessionList.Select(session => new StormSessionViewModel()
               {
                   Id = session.Id,
                   DateCreated = session.DateCreated,
                   Name = session.Name,
                   IdeaCount = session.Ideas.Count
               });

               return View(model);
           }

           public class NewSessionModel
           {
               [Required]
               public string SessionName { get; set; }
           }

           [HttpPost]
           public async Task<IActionResult> Index(NewSessionModel model)
           {
               if (!ModelState.IsValid)
               {
                   return BadRequest(ModelState);
               }

               await _sessionRepository.AddAsync(new BrainstormSession()
               {
                   DateCreated = DateTimeOffset.Now,
                   Name = model.SessionName
               });

               return RedirectToAction("Index");
           }
       }
   }

   ````

The controller is following the [explicit dependencies principle](http://deviq.com/explicit-dependencies-principle/), expecting dependency injection to provide it with an instance of `IBrainstormSessionRepository`. This makes it fairly easy to test using a mock object framework, like [Moq](https://www.nuget.org/packages/Moq/). The `HTTP GET Index` method has no looping or branching and only calls one method. To test this `Index` method, we need to verify that a `ViewResult` is returned, with a `ViewModel` from the repository's `List` method.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [17, 18], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/controllers/testing/sample/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/HomeControllerTests.cs"} -->

````c#

   using System;
   using System.Collections.Generic;
   using System.Linq;
   using System.Threading.Tasks;
   using Microsoft.AspNetCore.Mvc;
   using Moq;
   using TestingControllersSample.Controllers;
   using TestingControllersSample.Core.Interfaces;
   using TestingControllersSample.Core.Model;
   using TestingControllersSample.ViewModels;
   using Xunit;

   namespace TestingControllersSample.Tests.UnitTests
   {
       public class HomeControllerTests
       {
           [Fact]
           public async Task Index_ReturnsAViewResult_WithAListOfBrainstormSessions()
           {
               // Arrange
               var mockRepo = new Mock<IBrainstormSessionRepository>();
               mockRepo.Setup(repo => repo.ListAsync()).Returns(Task.FromResult(GetTestSessions()));
               var controller = new HomeController(mockRepo.Object);

               // Act
               var result = await controller.Index();

               // Assert
               var viewResult = Assert.IsType<ViewResult>(result);
               var model = Assert.IsAssignableFrom<IEnumerable<StormSessionViewModel>>(
                   viewResult.ViewData.Model);
               Assert.Equal(2, model.Count());
           }
           private List<BrainstormSession> GetTestSessions()
           {
               var sessions = new List<BrainstormSession>();
               sessions.Add(new BrainstormSession()
               {
                   DateCreated = new DateTime(2016, 7, 2),
                   Id = 1,
                   Name = "Test One"
               });
               sessions.Add(new BrainstormSession()
               {
                   DateCreated = new DateTime(2016, 7, 1),
                   Id = 2,
                   Name = "Test Two"
               });
               return sessions;
           }
       }
   }
   ````

The `HomeController` `HTTP POST Index` method (shown above) should verify:

* The action method returns a Bad Request `ViewResult` with the appropriate data when `ModelState.IsValid` is `false`

* The `Add` method on the repository is called and a `RedirectToActionResult` is returned with the correct arguments when `ModelState.IsValid` is true.

Invalid model state can be tested by adding errors using `AddModelError` as shown in the first test below.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [8, 15, 16, 37, 38, 39], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/controllers/testing/sample/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/HomeControllerTests.cs"} -->

````c#

   [Fact]
   public async Task IndexPost_ReturnsBadRequestResult_WhenModelStateIsInvalid()
   {
       // Arrange
       var mockRepo = new Mock<IBrainstormSessionRepository>();
       mockRepo.Setup(repo => repo.ListAsync()).Returns(Task.FromResult(GetTestSessions()));
       var controller = new HomeController(mockRepo.Object);
       controller.ModelState.AddModelError("SessionName", "Required");
       var newSession = new HomeController.NewSessionModel();

       // Act
       var result = await controller.Index(newSession);

       // Assert
       var badRequestResult = Assert.IsType<BadRequestObjectResult>(result);
       Assert.IsType<SerializableError>(badRequestResult.Value);
   }

   [Fact]
   public async Task IndexPost_ReturnsARedirectAndAddsSession_WhenModelStateIsValid()
   {
       // Arrange
       var mockRepo = new Mock<IBrainstormSessionRepository>();
       mockRepo.Setup(repo => repo.AddAsync(It.IsAny<BrainstormSession>()))
           .Returns(Task.CompletedTask)
           .Verifiable();
       var controller = new HomeController(mockRepo.Object);
       var newSession = new HomeController.NewSessionModel()
       {
           SessionName = "Test Name"
       };

       // Act
       var result = await controller.Index(newSession);

       // Assert
       var redirectToActionResult = Assert.IsType<RedirectToActionResult>(result);
       Assert.Null(redirectToActionResult.ControllerName);
       Assert.Equal("Index", redirectToActionResult.ActionName);
       mockRepo.Verify();
   }

   ````

The first test confirms when `ModelState` is not valid, the same `ViewResult` is returned as for a `GET` request. Note that the test doesn't attempt to pass in an invalid model. That wouldn't work anyway since model binding isn't running (though an [integration test](xref:mvc/controllers/testing#integration-testing) would use exercise model binding). In this case, model binding is not being tested. These unit tests are only testing what the code in the action method does.

The second test verifies that when `ModelState` is valid, a new `BrainstormSession` is added (via the repository), and the method returns a `RedirectToActionResult` with the expected properties. Mocked calls that aren't called are normally ignored, but calling `Verifiable` at the end of the setup call allows it to be verified in the test. This is done with the call to `mockRepo.Verify`, which will fail the test if the expected method was not called.

Note: The Moq library used in this sample makes it easy to mix verifiable, or "strict", mocks with non-verifiable mocks (also called "loose" mocks or stubs). Learn more about [customizing Mock behavior with Moq](https://github.com/Moq/moq4/wiki/Quickstart#customizing-mock-behavior).

Another controller in the app displays information related to a particular brainstorming session. It includes some logic to deal with invalid id values:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [19, 20, 21, 22, 25, 26, 27, 28], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/controllers/testing/sample/TestingControllersSample/src/TestingControllersSample/Controllers/SessionController.cs"} -->

````c#

   using System.Threading.Tasks;
   using Microsoft.AspNetCore.Mvc;
   using TestingControllersSample.Core.Interfaces;
   using TestingControllersSample.ViewModels;

   namespace TestingControllersSample.Controllers
   {
       public class SessionController : Controller
       {
           private readonly IBrainstormSessionRepository _sessionRepository;

           public SessionController(IBrainstormSessionRepository sessionRepository)
           {
               _sessionRepository = sessionRepository;
           }

           public async Task<IActionResult> Index(int? id)
           {
               if (!id.HasValue)
               {
                   return RedirectToAction("Index", "Home");
               }

               var session = await _sessionRepository.GetByIdAsync(id.Value);
               if (session == null)
               {
                   return Content("Session not found.");
               }

               var viewModel = new StormSessionViewModel()
               {
                   DateCreated = session.DateCreated,
                   Name = session.Name,
                   Id = session.Id
               };

               return View(viewModel);
           }
       }
   }

   ````

The controller action has three cases to test, one for each `return` statement:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [27, 28, 29, 46, 47, 64, 65, 66, 67, 68], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/controllers/testing/sample/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/SessionControllerTests.cs"} -->

````c#

   using System;
   using System.Collections.Generic;
   using System.Linq;
   using System.Threading.Tasks;
   using Microsoft.AspNetCore.Mvc;
   using Moq;
   using TestingControllersSample.Controllers;
   using TestingControllersSample.Core.Interfaces;
   using TestingControllersSample.Core.Model;
   using TestingControllersSample.ViewModels;
   using Xunit;

   namespace TestingControllersSample.Tests.UnitTests
   {
       public class SessionControllerTests
       {
           [Fact]
           public async Task IndexReturnsARedirectToIndexHomeWhenIdIsNull()
           {
               // Arrange
               var controller = new SessionController(sessionRepository: null);

               // Act
               var result = await controller.Index(id: null);

               // Arrange
               var redirectToActionResult = Assert.IsType<RedirectToActionResult>(result);
               Assert.Equal("Home", redirectToActionResult.ControllerName);
               Assert.Equal("Index", redirectToActionResult.ActionName);
           }

           [Fact]
           public async Task IndexReturnsContentWithSessionNotFoundWhenSessionNotFound()
           {
               // Arrange
               int testSessionId = 1;
               var mockRepo = new Mock<IBrainstormSessionRepository>();
               mockRepo.Setup(repo => repo.GetByIdAsync(testSessionId))
                   .Returns(Task.FromResult((BrainstormSession)null));
               var controller = new SessionController(mockRepo.Object);

               // Act
               var result = await controller.Index(testSessionId);

               // Assert
               var contentResult = Assert.IsType<ContentResult>(result);
               Assert.Equal("Session not found.", contentResult.Content);
           }

           [Fact]
           public async Task IndexReturnsViewResultWithStormSessionViewModel()
           {
               // Arrange
               int testSessionId = 1;
               var mockRepo = new Mock<IBrainstormSessionRepository>();
               mockRepo.Setup(repo => repo.GetByIdAsync(testSessionId))
                   .Returns(Task.FromResult(GetTestSessions().FirstOrDefault(s => s.Id == testSessionId)));
               var controller = new SessionController(mockRepo.Object);

               // Act
               var result = await controller.Index(testSessionId);

               // Assert
               var viewResult = Assert.IsType<ViewResult>(result);
               var model = Assert.IsType<StormSessionViewModel>(viewResult.ViewData.Model);
               Assert.Equal("Test One", model.Name);
               Assert.Equal(2, model.DateCreated.Day);
               Assert.Equal(testSessionId, model.Id);
           }

           private List<BrainstormSession> GetTestSessions()
           {
               var sessions = new List<BrainstormSession>();
               sessions.Add(new BrainstormSession()
               {
                   DateCreated = new DateTime(2016, 7, 2),
                   Id = 1,
                   Name = "Test One"
               });
               sessions.Add(new BrainstormSession()
               {
                   DateCreated = new DateTime(2016, 7, 1),
                   Id = 2,
                   Name = "Test Two"
               });
               return sessions;
           }
       }
   }
   ````

The app exposes functionality as a web API (a list of ideas associated with a brainstorming session and a method for adding new ideas to a session):

<a name=ideas-controller></a>

<!-- literal_block {"ids": ["ideas-controller"], "names": ["ideas-controller"], "highlight_args": {"hl_lines": [21, 22, 27, 30, 31, 32, 33, 34, 35, 36, 41, 42, 46, 52, 65], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/controllers/testing/sample/TestingControllersSample/src/TestingControllersSample/Api/IdeasController.cs"} -->

````c#

   using System;
   using System.Linq;
   using System.Threading.Tasks;
   using Microsoft.AspNetCore.Mvc;
   using TestingControllersSample.ClientModels;
   using TestingControllersSample.Core.Interfaces;
   using TestingControllersSample.Core.Model;

   namespace TestingControllersSample.Api
   {
       [Route("api/ideas")]
       public class IdeasController : Controller
       {
           private readonly IBrainstormSessionRepository _sessionRepository;

           public IdeasController(IBrainstormSessionRepository sessionRepository)
           {
               _sessionRepository = sessionRepository;
           }

           [HttpGet("forsession/{sessionId}")]
           public async Task<IActionResult> ForSession(int sessionId)
           {
               var session = await _sessionRepository.GetByIdAsync(sessionId);
               if (session == null)
               {
                   return NotFound(sessionId);
               }

               var result = session.Ideas.Select(idea => new IdeaDTO()
               {
                   Id = idea.Id,
                   Name = idea.Name,
                   Description = idea.Description,
                   DateCreated = idea.DateCreated
               }).ToList();

               return Ok(result);
           }

           [HttpPost("create")]
           public async Task<IActionResult> Create([FromBody]NewIdeaModel model)
           {
               if (!ModelState.IsValid)
               {
                   return BadRequest(ModelState);
               }

               var session = await _sessionRepository.GetByIdAsync(model.SessionId);
               if (session == null)
               {
                   return NotFound(model.SessionId);
               }

               var idea = new Idea()
               {
                   DateCreated = DateTimeOffset.Now,
                   Description = model.Description,
                   Name = model.Name
               };
               session.AddIdea(idea);

               await _sessionRepository.UpdateAsync(session);

               return Ok(session);
           }
       }
   }
   ````

The `ForSession` method returns a list of `IdeaDTO` types. Avoid returning your business domain entities directly via API calls, since frequently they include more data than the API client requires, and they unnecessarily couple your app's internal domain model with the API you expose externally. Mapping between domain entities and the types you will return over the wire can be done manually (using a LINQ `Select` as shown here) or using a library like [AutoMapper](https://github.com/AutoMapper/AutoMapper)

The unit tests for the `Create` and `ForSession` API methods:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [18, 23, 29, 33, 38, 39, 43, 50, 58, 59, 68, 69, 70, 76, 77, 78], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/controllers/testing/sample/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs"} -->

````c#

   using System;
   using System.Collections.Generic;
   using System.Linq;
   using System.Threading.Tasks;
   using Microsoft.AspNetCore.Mvc;
   using Moq;
   using TestingControllersSample.Api;
   using TestingControllersSample.ClientModels;
   using TestingControllersSample.Core.Interfaces;
   using TestingControllersSample.Core.Model;
   using Xunit;

   namespace TestingControllersSample.Tests.UnitTests
   {
       public class ApiIdeasControllerTests
       {
           [Fact]
           public async Task Create_ReturnsBadRequest_GivenInvalidModel()
           {
               // Arrange & Act
               var mockRepo = new Mock<IBrainstormSessionRepository>();
               var controller = new IdeasController(mockRepo.Object);
               controller.ModelState.AddModelError("error","some error");

               // Act
               var result = await controller.Create(model: null);

               // Assert
               Assert.IsType<BadRequestObjectResult>(result);
           }

           [Fact]
           public async Task Create_ReturnsHttpNotFound_ForInvalidSession()
           {
               // Arrange
               int testSessionId = 123;
               var mockRepo = new Mock<IBrainstormSessionRepository>();
               mockRepo.Setup(repo => repo.GetByIdAsync(testSessionId))
                   .Returns(Task.FromResult((BrainstormSession)null));
               var controller = new IdeasController(mockRepo.Object);

               // Act
               var result = await controller.Create(new NewIdeaModel());

               // Assert
               Assert.IsType<NotFoundObjectResult>(result);
           }

           [Fact]
           public async Task Create_ReturnsNewlyCreatedIdeaForSession()
           {
               // Arrange
               int testSessionId = 123;
               string testName = "test name";
               string testDescription = "test description";
               var testSession = GetTestSession();
               var mockRepo = new Mock<IBrainstormSessionRepository>();
               mockRepo.Setup(repo => repo.GetByIdAsync(testSessionId))
                   .Returns(Task.FromResult(testSession));
               var controller = new IdeasController(mockRepo.Object);

               var newIdea = new NewIdeaModel()
               {
                   Description = testDescription,
                   Name = testName,
                   SessionId = testSessionId
               };
               mockRepo.Setup(repo => repo.UpdateAsync(testSession))
                   .Returns(Task.CompletedTask)
                   .Verifiable();

               // Act
               var result = await controller.Create(newIdea);

               // Assert
               var okResult = Assert.IsType<OkObjectResult>(result);
               var returnSession = Assert.IsType<BrainstormSession>(okResult.Value);
               mockRepo.Verify();
               Assert.Equal(2, returnSession.Ideas.Count());
               Assert.Equal(testName, returnSession.Ideas.LastOrDefault().Name);
               Assert.Equal(testDescription, returnSession.Ideas.LastOrDefault().Description);
           }

           private BrainstormSession GetTestSession()
           {
               var session = new BrainstormSession()
               {
                   DateCreated = new DateTime(2016, 7, 2),
                   Id = 1,
                   Name = "Test One"
               };

               var idea = new Idea() { Name = "One" };
               session.AddIdea(idea);
               return session;
           }
       }
   }
   ````

As stated previously, to test the behavior of the method when `ModelState` is invalid, add a model error to the controller as part of the test. Don't try to test model validation or model binding in your unit tests - just test your action method's behavior when confronted with a particular `ModelState` value.

The second test depends on the repository returning null, so the mock repository is configured to return null. There's no need to create a test database (in memory or otherwise) and construct a query that will return this result - it can be done in a single statement as shown.

The last test verifies that the repository's `Update` method is called. As we did previously, the mock is called with `Verifiable` and then the mocked repository's `Verify` method is called to confirm the verifiable method was executed. It's not a unit test responsibility to ensure that the `Update` method saved the data; that can be done with an integration test.

<a name=integration-testing></a>

  ## Integration Testing

[Integration testing](../../testing/integration-testing.md) is done to ensure separate modules within your app work correctly together. Generally, anything you can test with a unit test, you can also test with an integration test, but the reverse isn't true. However, integration tests tend to be much slower than unit tests. Thus, it's best to test whatever you can with unit tests, and use integration tests for scenarios that involve multiple collaborators.

Although they may still be useful, mock objects are rarely used in integration tests. In unit testing, mock objects are an effective way to control how collaborators outside of the unit being tested should behave for the purposes of the test. In an integration test, real collaborators are used to confirm the whole subsystem works together correctly.

  ### Application State

One important consideration when performing integration testing is how to set your app's state. Tests need to run independent of one another, and so each test should start with the app in a known state. If your app doesn't use a database or have any persistence, this may not be an issue. However, most real-world apps persist their state to some kind of data store, so any modifications made by one test could impact another test unless the data store is reset. Using the built-in `TestServer`, it's very straightforward to host ASP.NET Core apps within our integration tests, but that doesn't necessarily grant access to the data it will use. If you're using an actual database, one approach is to have the app connect to a test database, which your tests can access and ensure is reset to a known state before each test executes.

In this sample application, I'm using Entity Framework Core's InMemoryDatabase support, so I can't just connect to it from my test project. Instead, I expose an `InitializeDatabase` method from the app's `Startup` class, which I call when the app starts up if it's in the `Development` environment. My integration tests automatically benefit from this as long as they set the environment to `Development`. I don't have to worry about resetting the database, since the InMemoryDatabase is reset each time the app restarts.

The `Startup` class:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [19, 20, 38, 39, 47, 56], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/controllers/testing/sample/TestingControllersSample/src/TestingControllersSample/Startup.cs"} -->

````c#

   using System;
   using System.Linq;
   using System.Threading.Tasks;
   using Microsoft.AspNetCore.Builder;
   using Microsoft.AspNetCore.Hosting;
   using Microsoft.EntityFrameworkCore;
   using Microsoft.Extensions.DependencyInjection;
   using Microsoft.Extensions.Logging;
   using TestingControllersSample.Core.Interfaces;
   using TestingControllersSample.Core.Model;
   using TestingControllersSample.Infrastructure;

   namespace TestingControllersSample
   {
       public class Startup
       {
           public void ConfigureServices(IServiceCollection services)
           {
               services.AddDbContext<AppDbContext>(
                   optionsBuilder => optionsBuilder.UseInMemoryDatabase());

               services.AddMvc();

               services.AddScoped<IBrainstormSessionRepository,
                   EFStormSessionRepository>();
           }

           public void Configure(IApplicationBuilder app,
               IHostingEnvironment env,
               ILoggerFactory loggerFactory)
           {
               loggerFactory.AddConsole(LogLevel.Warning);

               if (env.IsDevelopment())
               {
                   app.UseDeveloperExceptionPage();

                   var repository = app.ApplicationServices.GetService<IBrainstormSessionRepository>();
                   InitializeDatabaseAsync(repository).Wait();
               }

               app.UseStaticFiles();

               app.UseMvcWithDefaultRoute();
           }

           public async Task InitializeDatabaseAsync(IBrainstormSessionRepository repo)
           {
               var sessionList = await repo.ListAsync();
               if (!sessionList.Any())
               {
                   await repo.AddAsync(GetTestSession());
               }
           }

           public static BrainstormSession GetTestSession()
           {
               var session = new BrainstormSession()
               {
                   Name = "Test Session 1",
                   DateCreated = new DateTime(2016, 8, 1)
               };
               var idea = new Idea()
               {
                   DateCreated = new DateTime(2016, 8, 1),
                   Description = "Totally awesome idea",
                   Name = "Awesome idea"
               };
               session.AddIdea(idea);
               return session;
           }
       }
   }

   ````

You'll see the `GetTestSession` method used frequently in the integration tests below.

  ### Accessing Views

Each integration test class configures the `TestServer` that will run the ASP.NET Core app. By default, `TestServer` hosts the web app in the folder where it's running - in this case, the test project folder. Thus, when you attempt to test controller actions that return `ViewResult`, you may see this error:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "none"} -->

````none

   The view 'Index' was not found. The following locations were searched:
   (list of locations)
   ````

To correct this issue, you need to configure the server's content root, so it can locate the views for the project being tested. This is done by a call to `UseContentRoot` in the in the `TestFixture` class, shown below:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [32, 35], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "text", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/controllers/testing/sample/TestingControllersSample/tests/TestingControllersSample.Tests/IntegrationTests/TestFixture.cs"} -->

````text

   using System;
   using System.IO;
   using System.Net.Http;
   using System.Reflection;
   using Microsoft.AspNetCore.Hosting;
   using Microsoft.AspNetCore.Mvc.ApplicationParts;
   using Microsoft.AspNetCore.Mvc.Controllers;
   using Microsoft.AspNetCore.Mvc.ViewComponents;
   using Microsoft.AspNetCore.TestHost;
   using Microsoft.Extensions.DependencyInjection;
   using Microsoft.Extensions.PlatformAbstractions;

   namespace TestingControllersSample.Tests.IntegrationTests
   {
       /// <summary>
       /// A test fixture which hosts the target project (project we wish to test) in an in-memory server.
       /// </summary>
       /// <typeparam name="TStartup">Target project's startup type</typeparam>
       public class TestFixture<TStartup> : IDisposable
       {
           private const string SolutionName = "TestingControllersSample.sln";
           private readonly TestServer _server;

           public TestFixture()
               : this(Path.Combine("src"))
           {
           }

           protected TestFixture(string solutionRelativeTargetProjectParentDir)
           {
               var startupAssembly = typeof(TStartup).GetTypeInfo().Assembly;
               var contentRoot = GetProjectPath(solutionRelativeTargetProjectParentDir, startupAssembly);

               var builder = new WebHostBuilder()
                   .UseContentRoot(contentRoot)
                   .ConfigureServices(InitializeServices)
                   .UseEnvironment("Development")
                   .UseStartup(typeof(TStartup));

               _server = new TestServer(builder);

               Client = _server.CreateClient();
               Client.BaseAddress = new Uri("http://localhost");
           }

           public HttpClient Client { get; }

           public void Dispose()
           {
               Client.Dispose();
               _server.Dispose();
           }

           protected virtual void InitializeServices(IServiceCollection services)
           {
               var startupAssembly = typeof(TStartup).GetTypeInfo().Assembly;

               // Inject a custom application part manager. Overrides AddMvcCore() because that uses TryAdd().
               var manager = new ApplicationPartManager();
               manager.ApplicationParts.Add(new AssemblyPart(startupAssembly));

               manager.FeatureProviders.Add(new ControllerFeatureProvider());
               manager.FeatureProviders.Add(new ViewComponentFeatureProvider());

               services.AddSingleton(manager);
           }

           /// <summary>
           /// Gets the full path to the target project path that we wish to test
           /// </summary>
           /// <param name="solutionRelativePath">
           /// The parent directory of the target project.
           /// e.g. src, samples, test, or test/Websites
           /// </param>
           /// <param name="startupAssembly">The target project's assembly.</param>
           /// <returns>The full path to the target project.</returns>
           private static string GetProjectPath(string solutionRelativePath, Assembly startupAssembly)
           {
               // Get name of the target project which we want to test
               var projectName = startupAssembly.GetName().Name;

               // Get currently executing test project path
               var applicationBasePath = PlatformServices.Default.Application.ApplicationBasePath;

               // Find the folder which contains the solution file. We then use this information to find the target
               // project which we want to test.
               var directoryInfo = new DirectoryInfo(applicationBasePath);
               do
               {
                   var solutionFileInfo = new FileInfo(Path.Combine(directoryInfo.FullName, SolutionName));
                   if (solutionFileInfo.Exists)
                   {
                       return Path.GetFullPath(Path.Combine(directoryInfo.FullName, solutionRelativePath, projectName));
                   }

                   directoryInfo = directoryInfo.Parent;
               }
               while (directoryInfo.Parent != null);

               throw new Exception($"Solution root could not be located using application root {applicationBasePath}.");
           }
       }
   }
   ````

The `TestFixture` class is responsible for configuring and creating the `TestServer`, setting up an `HttpClient` to communicate with the `TestServer`. Each of the integration tests uses the `Client` property to connect to the test server and make a request.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [20, 26, 29, 30, 31, 35, 38, 39, 40, 41, 44, 47, 48], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/controllers/testing/sample/TestingControllersSample/tests/TestingControllersSample.Tests/IntegrationTests/HomeControllerTests.cs"} -->

````c#

   using System;
   using System.Collections.Generic;
   using System.Net;
   using System.Net.Http;
   using System.Threading.Tasks;
   using Xunit;

   namespace TestingControllersSample.Tests.IntegrationTests
   {
       public class HomeControllerTests : IClassFixture<TestFixture<TestingControllersSample.Startup>>
       {
           private readonly HttpClient _client;

           public HomeControllerTests(TestFixture<TestingControllersSample.Startup> fixture)
           {
               _client = fixture.Client;
           }

           [Fact]
           public async Task ReturnsInitialListOfBrainstormSessions()
           {
               // Arrange - get a session known to exist
               var testSession = Startup.GetTestSession();

               // Act
               var response = await _client.GetAsync("/");

               // Assert
               response.EnsureSuccessStatusCode();
               var responseString = await response.Content.ReadAsStringAsync();
               Assert.True(responseString.Contains(testSession.Name));
           }

           [Fact]
           public async Task PostAddsNewBrainstormSession()
           {
               // Arrange
               string testSessionName = Guid.NewGuid().ToString();
               var data = new Dictionary<string, string>();
               data.Add("SessionName", testSessionName);
               var content = new FormUrlEncodedContent(data);

               // Act
               var response = await _client.PostAsync("/", content);

               // Assert
               Assert.Equal(HttpStatusCode.Redirect, response.StatusCode);
               Assert.Equal("/", response.Headers.Location.ToString());
           }
       }
   }
   ````

In the first test above, the `responseString` holds the actual rendered HTML from the View, which can be inspected to confirm it contains expected results.

The second test constructs a form POST with a unique session name and POSTs it to the app, then verifies that the expected redirect is returned.

  ### API Methods

If your app exposes web APIs, it's a good idea to have automated tests confirm they execute as expected. The built-in `TestServer` makes it easy to test web APIs. If your API methods are using model binding, you should always check `ModelState.IsValid`, and integration tests are the right place to confirm that your model validation is working properly.

The following set of tests target the `Create` method in the [IdeasController](xref:mvc/controllers/testing#ideas-controller) class shown above:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/controllers/testing/sample/TestingControllersSample/tests/TestingControllersSample.Tests/IntegrationTests/ApiIdeasControllerTests.cs"} -->

````c#

   using System;
   using System.Collections.Generic;
   using System.Linq;
   using System.Net;
   using System.Net.Http;
   using System.Threading.Tasks;
   using Newtonsoft.Json;
   using TestingControllersSample.ClientModels;
   using TestingControllersSample.Core.Model;
   using Xunit;

   namespace TestingControllersSample.Tests.IntegrationTests
   {
       public class ApiIdeasControllerTests : IClassFixture<TestFixture<TestingControllersSample.Startup>>
       {
           internal class NewIdeaDto
           {
               public NewIdeaDto(string name, string description, int sessionId)
               {
                   Name = name;
                   Description = description;
                   SessionId = sessionId;
               }

               public string Name { get; set; }
               public string Description { get; set; }
               public int SessionId { get; set; }
           }

           private readonly HttpClient _client;

           public ApiIdeasControllerTests(TestFixture<TestingControllersSample.Startup> fixture)
           {
               _client = fixture.Client;
           }

           [Fact]
           public async Task CreatePostReturnsBadRequestForMissingNameValue()
           {
               // Arrange
               var newIdea = new NewIdeaDto("", "Description", 1);

               // Act
               var response = await _client.PostAsJsonAsync("/api/ideas/create", newIdea);

               // Assert
               Assert.Equal(HttpStatusCode.BadRequest, response.StatusCode);
           }

           [Fact]
           public async Task CreatePostReturnsBadRequestForMissingDescriptionValue()
           {
               // Arrange
               var newIdea = new NewIdeaDto("Name", "", 1);

               // Act
               var response = await _client.PostAsJsonAsync("/api/ideas/create", newIdea);

               // Assert
               Assert.Equal(HttpStatusCode.BadRequest, response.StatusCode);
           }

           [Fact]
           public async Task CreatePostReturnsBadRequestForSessionIdValueTooSmall()
           {
               // Arrange
               var newIdea = new NewIdeaDto("Name", "Description", 0);

               // Act
               var response = await _client.PostAsJsonAsync("/api/ideas/create", newIdea);

               // Assert
               Assert.Equal(HttpStatusCode.BadRequest, response.StatusCode);
           }

           [Fact]
           public async Task CreatePostReturnsBadRequestForSessionIdValueTooLarge()
           {
               // Arrange
               var newIdea = new NewIdeaDto("Name", "Description", 1000001);

               // Act
               var response = await _client.PostAsJsonAsync("/api/ideas/create", newIdea);

               // Assert
               Assert.Equal(HttpStatusCode.BadRequest, response.StatusCode);
           }

           [Fact]
           public async Task CreatePostReturnsNotFoundForInvalidSession()
           {
               // Arrange
               var newIdea = new NewIdeaDto("Name", "Description", 123);

               // Act
               var response = await _client.PostAsJsonAsync("/api/ideas/create", newIdea);

               // Assert
               Assert.Equal(HttpStatusCode.NotFound, response.StatusCode);
           }

           [Fact]
           public async Task CreatePostReturnsCreatedIdeaWithCorrectInputs()
           {
               // Arrange
               var testIdeaName = Guid.NewGuid().ToString();
               var newIdea = new NewIdeaDto(testIdeaName, "Description", 1);

               // Act
               var response = await _client.PostAsJsonAsync("/api/ideas/create", newIdea);

               // Assert
               response.EnsureSuccessStatusCode();
               var returnedSession = await response.Content.ReadAsJsonAsync<BrainstormSession>();
               Assert.Equal(2, returnedSession.Ideas.Count);
               Assert.True(returnedSession.Ideas.Any(i => i.Name == testIdeaName));
           }

           [Fact]
           public async Task ForSessionReturnsNotFoundForBadSessionId()
           {
               // Arrange & Act
               var response = await _client.GetAsync("/api/ideas/forsession/500");

               // Assert
               Assert.Equal(HttpStatusCode.NotFound, response.StatusCode);
           }

           [Fact]
           public async Task ForSessionReturnsIdeasForValidSessionId()
           {
               // Arrange
               var testSession = Startup.GetTestSession();

               // Act
               var response = await _client.GetAsync("/api/ideas/forsession/1");

               // Assert
               response.EnsureSuccessStatusCode();
               var ideaList = JsonConvert.DeserializeObject<List<IdeaDTO>>(
                   await response.Content.ReadAsStringAsync());
               var firstIdea = ideaList.First();
               Assert.Equal(testSession.Ideas.First().Name, firstIdea.Name);
           }
       }
   }
   ````

Unlike integration tests of actions that returns HTML views, web API methods that return results can usually be deserialized as strongly typed objects, as the last test above shows. In this case, the test deserializes the result to a `BrainstormSession` instance, and confirms that the idea was correctly added to its collection of ideas.

You'll find additional examples of integration tests in this article's [sample project](https://github.com/aspnet/Docs/tree/master/aspnet/mvc/controllers/testing/sample).
