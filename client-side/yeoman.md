---
uid: client-side/yeoman
---
  # Building Projects with Yeoman

By [Steve Smith](http://ardalis.com), [Scott Addie](https://scottaddie.com), [Rick Anderson](https://twitter.com/RickAndMSFT), [Noel Rice](http://blog.falafel.com/author/noel-rice/), and [Shayne Boyer](https://twitter.com/spboyer)

[Yeoman](http://yeoman.io/) generates complete projects for a given set of client tools. Yeoman is an open-source tool that works like a Visual Studio project template. The Yeoman command line tool [yo](https://github.com/yeoman/yo) works alongside a Yeoman generator. Generators define the technologies that go into a project.

  ## Install Node.js, npm, and Yeoman

To get started with Yeoman install [Node.js](https://nodejs.org/en/). The installer includes [Node.js](https://nodejs.org/en/) and [npm](https://www.npmjs.com/).

Follow the instructions on [http://yeoman.io/learning/](http://yeoman.io/learning/) to install [yo](https://github.com/yeoman/yo), bower, grunt, and gulp.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "console"} -->

````console

   npm install -g yo bower
   ````

Note: If you get the error `npm ERR! Please try running this command again as root/Administrator.` on Mac OS, run the following command using [sudo](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man8/sudo.8.html): `sudo npm install -g yo bower grunt-cli gulp`

From the command line, install the ASP.NET generator:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "console"} -->

````console

   npm install -g generator-aspnet
   ````

Note: If you get a permission error, run the command under `sudo` as described above.

The `–g` flag installs the generator globally, so that it can be used from any path.

  ## Create an ASP.NET app

Create a directory for your projects

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "console"} -->

````console

   mkdir src
   cd src
   ````

Run the ASP.NET generator for `yo`

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "console"} -->

````console

   yo aspnet
   ````

The generator displays a menu. Arrow down to the **Empty Web Application** project and tap **Enter**:

![image](yeoman/_static/yeoman-yo-aspnet.png)

Use "EmptyWeb1" for the app name and then tap **Enter**

Yeoman will scaffold the project and its supporting files. Suggested next steps are also provided in the form of commands.

![image](yeoman/_static/yeoman-yo-aspnet-created.png)

The [ASP.NET generator](https://www.npmjs.com/package/generator-aspnet) creates ASP.NET Core projects that can be loaded into Visual Studio Code, Visual Studio, or run from the command line.

  ## Restore, build and run

Follow the suggested commands by changing directories to the `EmptyWeb1` directory. Then run `dotnet restore`.

![image](yeoman/_static/dotnet-restore.png)

Build and run the app using `dotnet build` and `dotnet run`:

![image](yeoman/_static/dotnet-build-run.png)

At this point you can navigate to the URL shown to test the newly created ASP.NET Core app.

  ## Client-Side Packages

The front end resources are provided by the templates from the yeoman generator using the [Bower](bower.md) client-side package manager, adding *bower.json* and *.bowerrc* files to restore client-side packages using the [Bower](bower.md) client-side package manager.

The [BundlerMinifier](https://github.com/madskristensen/BundlerMinifier/wiki) component is also included by default for ease of concatenation (bundling) and minification of CSS, JavaScript and HTML.

  ## Building and Running from Visual Studio

You can load your generated ASP.NET Core web project directly into Visual Studio, then build and run your project from there. Follow the instructions above to scaffold a new ASP.NET Core app using yeoman. This time, choose **Web Application** from the menu and name the app `MyWebApp`.

Open Visual Studio. From the File menu, select Open ‣ Project/Solution.

In the Open Project dialog, navigate to the *project.json* file, select it, and click the **Open** button. In the Solution Explorer, the project should look something like the screenshot below.

![image](yeoman/_static/yeoman-solution.png)

Yeoman scaffolds a MVC web application, complete with both server- and client-side build support. Server-side dependencies are listed under the **References** node, and client-side dependencies in the **Dependencies** node of Solution Explorer. Dependencies are restored automatically when the project is loaded.

![image](yeoman/_static/yeoman-loading-dependencies.png)

When all the dependencies are restored, press **F5** to run the project. The default home page displays in the browser.

![image](yeoman/_static/yeoman-home-page.png)

  ## Restoring, Building, and Hosting from the Command Line

You can prepare and host your web application using the [.NET Core](https://microsoft.com/net/core) command-line interface.

From the command line, change the current directory to the folder containing the project (that is, the folder containing the *project.json* file):

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "console"} -->

````console

   cd src\MyWebApp
   ````

From the command line, restore the project's NuGet package dependencies:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "console"} -->

````console

   dotnet restore
   ````

Run the application:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "console"} -->

````console

   dotnet run
   ````

The cross-platform [Kestrel](../fundamentals/servers.md#kestrel.md) web server will begin listening on port 5000.

Open a web browser, and navigate to http://localhost:5000.

![image](yeoman/_static/yeoman-home-page_5000.png)

  ## Adding to Your Project with Sub Generators

You can add new generated files using Yeoman even after the project is created. Use [sub generators](https://www.github.com/omnisharp/generator-aspnet#sub-generators) to add any of the file types that make up your project. For example, to add a new class to your project, enter the `yo aspnet:Class` command followed by the name of the class. Execute the following command from the directory in which the file should be created:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "console"} -->

````console

   yo aspnet:Class Person
   ````

The result is a file named Person.cs with a class named `Person`:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#"} -->

````c#

   using System;
   using System.Collections.Generic;
   using System.Linq;
   using System.Threading.Tasks;

   namespace MyNamespace
   {
       public class Person
       {
           public Person()
           {
           }
       }
   }
   ````

  ## Related Resources

* [Servers (HttpPlatformHandler, Kestrel and WebListener)](../fundamentals/servers.md)

* [Your First ASP.NET Core Application on a Mac Using Visual Studio Code](../tutorials/your-first-mac-aspnet.md)

* [Fundamentals](../fundamentals/index.md)
