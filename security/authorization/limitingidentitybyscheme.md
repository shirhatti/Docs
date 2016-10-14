---
uid: security/authorization/limitingidentitybyscheme
---
<a name=security-authorization-limiting-by-scheme></a>

  # Limiting identity by scheme

In some scenarios, such as Single Page Applications it is possible to end up with multiple authentication methods. For example, your application may use cookie-based authentication to log in and bearer authentication for JavaScript requests. In some cases you may have multiple instances of an authentication middleware. For example, two cookie middlewares where one contains a basic identity and one is created when a multi-factor authentication has triggered because the user requested an operation that requires extra security.

Authentication schemes are named when authentication middleware is configured during authentication, for example

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#"} -->

````c#

   app.UseCookieAuthentication(new CookieAuthenticationOptions()
   {
       AuthenticationScheme = "Cookie",
       LoginPath = new PathString("/Account/Unauthorized/"),
       AccessDeniedPath = new PathString("/Account/Forbidden/"),
       AutomaticAuthenticate = false
   });

   app.UseBearerAuthentication(options =>
   {
       options.AuthenticationScheme = "Bearer";
       options.AutomaticAuthenticate = false;
   });
   ````

In this configuration two authentication middlewares have been added, one for cookies and one for bearer.

   Note: When adding multiple authentication middleware you should ensure that no middleware is configured to run automatically. You do this by setting the `AutomaticAuthenticate` options property to false. If you fail to do this filtering by scheme will not work.

  ## Selecting the scheme with the Authorize attribute

As no authentication middleware is configured to automatically run and create an identity you must, at the point of authorization choose which middleware will be used. The simplest way to select the middleware you wish to authorize with is to use the [ActiveAuthenticationSchemes](http://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Authorization/AuthorizeAttribute/index.html.md#Microsoft.AspNetCore.Authorization.AuthorizeAttribute.ActiveAuthenticationSchemes.md) property. This property accepts a comma delimited list of Authentication Schemes to use. For example;

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#"} -->

````c#

   [Authorize(ActiveAuthenticationSchemes = "Cookie,Bearer")]
   public class MixedController : Controller
   ````

In the example above both the cookie and bearer middlewares will run and have a chance to create and append an identity for the current user. By specifying a single scheme only the specified middleware will run;

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#"} -->

````c#

   [Authorize(ActiveAuthenticationSchemes = "Bearer")]
   ````

In this case only the middleware with the Bearer scheme would run, and any cookie based identities would be ignored.

  ## Selecting the scheme with policies

If you prefer to specify the desired schemes in [policy](policies.md#security-authorization-policies-based.md) you can set the [AuthenticationSchemes](http://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Authorization/AuthorizationPolicyBuilder/index.html.md#Microsoft.AspNetCore.Authorization.AuthorizationPolicyBuilder.AuthenticationSchemes.md) collection when adding your policy.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#"} -->

````c#

   options.AddPolicy("Over18", policy =>
   {
       policy.AuthenticationSchemes.Add("Bearer");
       policy.RequireAuthenticatedUser();
       policy.Requirements.Add(new Over18Requirement());
   });
   ````

In this example the Over18 policy will only run against the identity created by the `Bearer` middleware.
