---
uid: security/data-protection/compatibility/cookie-sharing
---
  # Sharing cookies between applications

Web sites commonly consist of many individual web applications, all working together harmoniously. If an application developer wants to provide a good single-sign-on experience, he'll often need all of the different web applications within the site to share authentication tickets between each other.

To support this scenario, the data protection stack allows sharing Katana cookie authentication and ASP.NET Core cookie authentication tickets.

  ## Sharing authentication cookies between applications

To share authentication cookies between two different ASP.NET Core applications, configure each application that should share cookies as follows.

In your configure method use the CookieAuthenticationOptions to set up the data protection service for cookies and the AuthenticationScheme to match ASP.NET 4.X.

If you're using identity:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#"} -->

````c#

   app.AddIdentity<ApplicationUser, IdentityRole>(options =>
   {
       options.Cookies.ApplicationCookie.AuthenticationScheme = "ApplicationCookie";
       options.Cookies.ApplicationCookie.DataProtectionProvider = DataProtectionProvider.Create(new DirectoryInfo(@"c:\shared-auth-ticket-keys\"));
   });
   ````

If you're using cookies directly:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#"} -->

````c#

   app.UseCookieAuthentication(new CookieAuthenticationOptions
   {
       DataProtectionProvider = DataProtectionProvider.Create(new DirectoryInfo(@"c:\shared-auth-ticket-keys\"))
   });
   ````

When used in this manner, the DirectoryInfo should point to a key storage location specifically set aside for authentication cookies. The cookie authentication middleware will use the explicitly provided implementation of the DataProtectionProvider, which is now isolated from the data protection system used by other parts of the application. The application name is ignored (intentionally so, since you're trying to get multiple applications to share payloads).

Caution: You should consider configuring the DataProtectionProvider such that keys are encrypted at rest, as in the below example.

  <!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#"} -->

  ````c#

     app.UseCookieAuthentication(new CookieAuthenticationOptions
     {
         DataProtectionProvider = DataProtectionProvider.Create(
             new DirectoryInfo(@"c:\shared-auth-ticket-keys\"),
             configure =>
             {
                 configure.ProtectKeysWithCertificate("thumbprint");
             })
     });
     ````

  ## Sharing authentication cookies between ASP.NET 4.x and ASP.NET Core applications

ASP.NET 4.x applications which use Katana cookie authentication middleware can be configured to generate authentication cookies which are compatible with the ASP.NET Core cookie authentication middleware. This allows upgrading a large site's individual applications piecemeal while still providing a smooth single sign on experience across the site.

Tip: You can tell if your existing application uses Katana cookie authentication middleware by the existence of a call to UseCookieAuthentication in your project's Startup.Auth.cs. ASP.NET 4.x web application projects created with Visual Studio 2013 and later use the Katana cookie authentication middleware by default.

Note: Your ASP.NET 4.x application must target .NET Framework 4.5.1 or higher, otherwise the necessary NuGet packages will fail to install.

To share authentication cookies between your ASP.NET 4.x applications and your ASP.NET Core applications, configure the ASP.NET Core application as stated above, then configure your ASP.NET 4.x applications by following the steps below.

1. Install the package Microsoft.Owin.Security.Interop into each of your ASP.NET 4.x applications.

2. In Startup.Auth.cs, locate the call to UseCookieAuthentication, which will generally look like the following.

   <!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#"} -->

   ````c#

      app.UseCookieAuthentication(new CookieAuthenticationOptions
      {
          // ...
      });
      ````

3. Modify the call to UseCookieAuthentication as follows, changing the CookieName to match the name used by the ASP.NET Core cookie authentication middleware, and providing an instance of a DataProtectionProvider that has been initialized to a key storage location.

   <!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#"} -->

   ````c#

      app.UseCookieAuthentication(new CookieAuthenticationOptions
      {
          AuthenticationType = DefaultAuthenticationTypes.ApplicationCookie,
          CookieName = ".AspNetCore.Cookies",
          // CookiePath = "...", (if necessary)
          // ...
          TicketDataFormat = new AspNetTicketDataFormat(
              new DataProtectorShim(
                  DataProtectionProvider.Create(new DirectoryInfo(@"c:\shared-auth-ticket-keys\"))
                  .CreateProtector("Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationMiddleware",
                  "Cookies", "v2")))
      });
      ````

   The DirectoryInfo has to point to the same storage location that you pointed your ASP.NET Core application to and should be configured using the same settings.

The ASP.NET 4.x and ASP.NET Core applications are now configured to share authentication cookies.

Note: You'll need to make sure that the identity system for each application is pointed at the same user database. Otherwise the identity system will produce failures at runtime when it tries to match the information in the authentication cookie against the information in its database.
