---
uid: security/authorization/simple
---
<a name=security-authorization-simple></a>

  # Simple Authorization

Authorization in MVC is controlled through the [AuthorizeAttribute](http://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Authorization/AuthorizeAttribute/index.html.md#Microsoft.AspNetCore.Authorization.AuthorizeAttribute.md) attribute and its various parameters. At its simplest applying the [AuthorizeAttribute](http://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Authorization/AuthorizeAttribute/index.html.md#Microsoft.AspNetCore.Authorization.AuthorizeAttribute.md) attribute to a controller or action limits access to the controller or action to any authenticated user.

For example, the following code limits access to the `AccountController` to any authenticated user.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#"} -->

````c#

   [Authorize]
   public class AccountController : Controller
   {
       public ActionResult Login()
       {
       }

       public ActionResult Logout()
       {
       }
   }
   ````

If you want to apply authorization to an action rather than the controller simply apply the [AuthorizeAttribute](http://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Authorization/AuthorizeAttribute/index.html.md#Microsoft.AspNetCore.Authorization.AuthorizeAttribute.md) attribute to the action itself;

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#"} -->

````c#

   public class AccountController : Controller
   {
       public ActionResult Login()
       {
       }

       [Authorize]
       public ActionResult Logout()
       {
       }
   }
   ````

Now only authenticated users can access the logout function.

You can also use the [AllowAnonymousAttribute](http://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Authorization/AllowAnonymousAttribute/index.html.md#Microsoft.AspNetCore.Authorization.AllowAnonymousAttribute.md) attribute to allow access by non-authenticated users to individual actions; for example

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#"} -->

````c#

   [Authorize]
   public class AccountController : Controller
   {
       [AllowAnonymous]
       public ActionResult Login()
       {
       }

       public ActionResult Logout()
       {
       }
   }
   ````

This would allow only authenticated users to the `AccountController`, except for the `Login` action, which is accessible by everyone, regardless of their authenticated or unauthenticated / anonymous status.

Warning: `[AllowAnonymous]` bypasses all authorization statements. If you apply combine `[AllowAnonymous]` and any `[Authorize]` attribute then the Authorize attributes will always be ignored. For example if you apply `[AllowAnonymous]` at the controller level any `[Authorize]` attributes on the same controller, or on any action within it will be ignored.
