Working with Static Files
=========================
By `Tom Archer`

Static files, which include HTML files, CSS files, image files, and JavaScript files, are assets that the app will serve directly to clients. In this article, we'll cover the following topics as they relate to ASP.NET 5 and static files.

In this article:
  - `Serving static files`_
  - `Enabling directory browsing`_
  - `Serving default files`_
  - `Using the UseFileServer method`_
  - `Best practices`_

Serving static files
--------------------

By default, static files are stored in the `webroot` of your project. The location of the webroot is defined in the project's ``project.json`` file where the default is `wwwroot`.

.. literalinclude:: /../common/samples/WebApplication1/src/WebApplication1/project.json
  :linenos:
  :language: json
  :lines: 2

Static files can be stored in any folder under the webroot and accessed with a relative path to that root. For example, when you create a default Web application project using Visual Studio, there are several folders created within the webroot folder - ``css``, ``images``, and ``js``. In order to directly access an image in the ``images`` subfolder, the URL would look like the following:

  http://<yourApp>/images/<imageFileName>

In order for static files to be served, you must configure the :doc:`middleware` to add static files to the pipeline. This is accomplished by calling the  ``UseStaticFiles`` extension method from  ``Startup.Configure`` as follows:

.. code-block:: c#
  :emphasize-lines: 5

  public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
  {
    ...
    // Add static files to the request pipeline.
    app.UseStaticFiles();
    ...

Enabling directory browsing
---------------------------

Directory browsing allows the user of your Web app to see a list of directories and files within a specified directory (including the root). By default, this functionality is turned off such that if the user attempts to display a directory within an ASP.NET Web app, the browser displays an error. To enable directory browsing for your Web app, call the ``UseDirectoryBrowser`` extension method from  ``Startup.Configure`` as follows:

.. code-block:: c#
  :emphasize-lines: 5

  public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
  {
    ...
    // Turn on directory browsing for the current directory.
    app.UseDirectoryBrowser();
    ...

The following figure illustrates the results of browsing to the Web app's ``images`` folder with directory browsing turned on:

.. image:: static-files/_static/dir-browse.png


Serving default files
---------------------

In order for your Web app to serve a default page without the user having to fully qualify the URI, call the ``UseDefaultFiles`` extension method from ``Startup.Configure`` as follows. Note that you must still call ``UseStaticFiles`` as well. This is because ``UseDefaultFiles`` is a `URL re-writer` that doesn't actually serve the file. You must still specify middleware (``UseStaticFiles``, in this case) to serve the file.

.. code-block:: c#
  :emphasize-lines: 5-6

  public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
  {
    ...
    // Serve the default file, if present.
    app.UseDefaultFiles();
    app.UseStaticFiles();
    ...

If you call the ``'UseDefaultFiles`` extension method and the user enters a URI of a folder, the middleware will search (in order) for one of the following files. If one of these files is found, that file will be used as if the user had entered the fully qualified URI (although the browser URL will continue to show the URI entered by the user).

  - default.htm
  - default.html
  - index.htm
  - index.html

To specify a different default file from the ones listed above, instantiate a ``DefaultFilesOptions`` object and set its ``DefaultFileNames`` string list to an appropriate list of names for your app. Then, call one of the overloaded ``UseDefaultFiles`` methods passing it the ``DefaultFilesOptions`` object. The following example code removes all of the default files from the ``DefaultFileNames`` list and adds the ``mydefault.html`` as the only default file for which to search.

.. code-block:: c#
  :emphasize-lines: 5-8

  public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
  {
    ...
    // Serve my app-specific default file, if present.
    DefaultFilesOptions options = new DefaultFilesOptions();
    options.DefaultFileNames.Clear();
    options.DefaultFileNames.Add("mydefault.html");
    app.UseDefaultFiles(options);
    app.UseStaticFiles();
    ...

Using the UseFileServer method
------------------------------

In addition to the ``UseStaticFiles``, ``UseDefaultFiles``, and ``UseDirectoryBrowser`` extensions methods, there is also a single method - ``UseFileServer`` - that combines the functionality of all three methods. The following example code shows some common ways to use this method:

.. code-block:: c#

  // Add static files, default files, and directory browsing to the request pipeline.
  app.UseFileServer(true);

.. code-block:: c#

  // Add static files, default files, but NOT directory browsing to the request pipeline.
  app.UseFileServer();

Best practices
--------------

This section includes a list of best practices for working with static files:
  - Code files (including C# and Razor files) should be placed outside of the app project's webroot. This creates a clean separation between your app's static (non-compilable) content and source code.

Summary
-------
In this article, you learned how the static files middleware component in ASP.NET 5 allows you to serve static files, enable directory browsing, and serve default files.

Additional Resources
--------------------

- :doc:`middleware`
