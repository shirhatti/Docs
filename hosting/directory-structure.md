---
uid: hosting/directory-structure
---
<a name=directory-structure></a>

  # Directory Structure

By [Luke Latham](https://github.com/GuardRex)

In ASP.NET Core, the application directory, *publish*, is comprised of application files, config files, static assets, packages, and the runtime (for self-contained apps). This is the same directory structure as previous versions of ASP.NET, where the entire application lives inside the web root directory.

<!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- table -->

* Indicates a directory

The contents of the *publish* directory represent the *content root path*, also called the *application base path*, of the deployment. Whatever name is given to the *publish* directory in the deployment, its location serves as the server's physical path to the hosted application. The *wwwroot* directory, if present, only contains static assets. The *logs* directory may be included in the deployment by creating it in the project and adding it to **publishOptions** of *project.json* or by physically creating the directory on the server.

The deployment directory requires Read/Execute permissions, while the *logs* directory requires Read/Write permissions. Additional directories where assets will be written require Read/Write permissions.
