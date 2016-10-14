---
uid: mvc/views/working-with-forms
---
Warning: This page documents version 1.0.0-rc1 and has not yet been updated for version 1.0.0

  # Working with Forms

By [Rick Anderson](https://twitter.com/RickAndMSFT), [Dave Paquette](https://twitter.com/Dave_Paquette) and [Jerrie Pelser](https://twitter.com/jerriepelser)

This document demonstrates working with Forms and the HTML elements commonly used on a Form. The HTML [Form](https://www.w3.org/TR/html401/interact/forms.html) element provides the primary mechanism web apps use to post back data to the server. Most of this document describes [Tag Helpers](tag-helpers/intro.md) and how they can help you productively create robust HTML forms. We recommend you read [Introduction to Tag Helpers](tag-helpers/intro.md) before you read this document.

In many cases, [HTML Helpers](html-helpers.md) provide an alternative approach to a specific Tag Helper, but it's important to recognize that Tag Helpers do not replace HTML Helpers and there is not a Tag Helper for each HTML Helper. When an HTML Helper alternative exists, it is mentioned.

<a name=my-asp-route-param-ref-label></a>

  ## The Form Tag Helper

The [Form](https://www.w3.org/TR/html401/interact/forms.html) Tag Helper:

* Generates the HTML [<FORM>](https://www.w3.org/TR/html401/interact/forms.html) `action` attribute value for a MVC controller action or named route

* Generates a hidden [Request Verification Token](http://www.asp.net/mvc/overview/security/xsrfcsrf-prevention-in-aspnet-mvc-and-web-pages) to prevent cross-site request forgery (when used with the `[ValidateAntiForgeryToken]` attribute in the HTTP Post action method)

* Provides the `asp-route-<Parameter Name>` attribute, where `<Parameter Name>` is added to the route values. The  `routeValues` parameters to `Html.BeginForm` and `Html.BeginRouteForm` provide similar functionality.

* Has an HTML Helper alternative `Html.BeginForm` and `Html.BeginRouteForm`

Sample:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/Views/Demo/RegisterFormOnly.cshtml"} -->

````HTML

   <form asp-controller="Demo" asp-action="Register" method="post">
       <!-- Input and Submit elements -->
   </form>
   ````

The Form Tag Helper above generates the following HTML:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML"} -->

````HTML

   <form method="post" action="/Demo/Register">
     <!-- Input and Submit elements -->
     <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>" />
    </form>
   ````

The MVC runtime generates the `action` attribute value from the Form Tag Helper attributes `asp-controller` and `asp-action`. The Form Tag Helper also generates a hidden [Request Verification Token](http://www.asp.net/mvc/overview/security/xsrfcsrf-prevention-in-aspnet-mvc-and-web-pages) to prevent cross-site request forgery (when used with the `[ValidateAntiForgeryToken]` attribute in the HTTP Post action method). Protecting a pure HTML Form from cross-site request forgery is very difficult, the Form Tag Helper provides this service for you.

  ### Using a named route

The `asp-route` Tag Helper attribute can also generate markup for the HTML `action` attribute. An app with a [route](../../fundamentals/routing.md)  named `register` could use the following markup for the registration page:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [4], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/Views/Demo/RegisterRoute.cshtml"} -->

````HTML

   <form asp-route="register" method="post">
       <!-- Input and Submit elements -->
   </form>
   ````

Many of the views in the *Views/Account* folder (generated when you create a new web app with *Individual User Accounts*) contain the [asp-route-returnurl](http://docs.asp.net/en/latest/mvc/views/working-with-forms.html#the-form-tag-helper) attribute:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [2]}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "none"} -->

````none

   <form asp-controller="Account" asp-action="Login"
     asp-route-returnurl="@ViewData["ReturnUrl"]"
     method="post" class="form-horizontal" role="form">
   ````

Note:
   With the built in templates, `returnUrl` is only populated automatically when you try to access an authorized resource but are not authenticated or authorized. When you attempt an unauthorized access, the security middleware redirects you to the login page with the `returnUrl` set.

  ## The Input Tag Helper

The Input Tag Helper binds an HTML [<input>](https://www.w3.org/wiki/HTML/Elements/input) element to a model expression in your razor view.

Syntax:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML"} -->

````HTML

   <input asp-for="<Expression Name>" />
   ````

The Input Tag Helper:

* Generates the `id` and `name` HTML attributes for the expression name specified in the `asp-for` attribute.  `asp-for="Property1.Property2"` is equivalent to `m => m.Property1.Property2`, that is the attribute value literally is part of an expression. The name of the expression is what's used for the `asp-for` attribute value.

* Sets the HTML `type` attribute value based on the model type and  [data annotation](https://msdn.microsoft.com/en-us/library/system.componentmodel.dataannotations.aspx) attributes applied to the model property

* Will not overwrite the HTML `type` attribute value when one is specified

* Generates [HTML5](https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/HTML5)  validation attributes from [data annotation](https://msdn.microsoft.com/en-us/library/system.componentmodel.dataannotations.aspx) attributes applied to model properties

* Has an HTML Helper feature overlap with `Html.TextBoxFor` and `Html.EditorFor`. See the **HTML Helper alternatives to Input Tag Helper** section for details.

* Provides strong typing. If the name of the property changes and you don't update the Tag Helper you'll get an error similar to the following:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML"} -->

````HTML

   An error occurred during the compilation of a resource required to process
   this request. Please review the following specific error details and modify
   your source code appropriately.

   Type expected
    'RegisterViewModel' does not contain a definition for 'Email' and no
    extension method 'Email' accepting a first argument of type 'RegisterViewModel'
    could be found (are you missing a using directive or an assembly reference?)
   ````

The `Input` Tag Helper sets the HTML `type` attribute based on the .NET type. The following table lists some common .NET types and generated HTML type (not every .NET type is listed).

<!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- table -->

The following table shows some common [data annotations](https://msdn.microsoft.com/en-us/library/system.componentmodel.dataannotations.aspx) attributes that the input tag helper will map to specific input types (not every validation attribute is listed):

<!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- table -->

Sample:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/ViewModels/RegisterViewModel.cs"} -->

````c#

   using System.ComponentModel.DataAnnotations;

   namespace FormsTagHelper.ViewModels
   {
       public class RegisterViewModel
       {
           [Required]
           [EmailAddress]
           [Display(Name = "Email Address")]
           public string Email { get; set; }

           [Required]
           [DataType(DataType.Password)]
           public string Password { get; set; }
       }
   }
   ````

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/Views/Demo/RegisterInput.cshtml"} -->

````HTML

   @model RegisterViewModel

   <form asp-controller="Demo" asp-action="RegisterInput" method="post">
       Email:  <input asp-for="Email" /> <br />
       Password: <input asp-for="Password" /><br />
       <button type="submit">Register</button>
   </form>
   ````

The code above generates the following HTML:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML"} -->

````HTML

     <form method="post" action="/Demo/RegisterInput">
       Email:
       <input type="email" data-val="true"
              data-val-email="The Email Address field is not a valid e-mail address."
              data-val-required="The Email Address field is required."
              id="Email" name="Email" value="" /> <br>
       Password:
       <input type="password" data-val="true"
              data-val-required="The Password field is required."
              id="Password" name="Password" /><br>
       <button type="submit">Register</button>
     <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>" />
   </form>
   ````

The data annotations applied to the `Email` and `Password` properties generate metadata on the model. The Input Tag Helper consumes the model metadata and produces [HTML5](https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/HTML5) `data-val-*` attributes (see [Model Validation](../models/validation.md)). These attributes describe the validators to attach to the input fields. This provides unobtrusive HTML5 and [jQuery](https://jquery.com/) validation. The unobtrusive attributes have the format `data-val-rule="Error Message"`, where rule is the name of the validation rule (such as `data-val-required`, `data-val-email`, `data-val-maxlength`, etc.) If an error message is provided in the attribute, it is displayed as the value for the `data-val-rule` attribute. There are also attributes of the form `data-val-ruleName-argumentName="argumentValue"` that provide additional details about the rule, for example, `data-val-maxlength-max="1024"` .

  ### HTML Helper alternatives to Input Tag Helper

`Html.TextBox`, `Html.TextBoxFor`, `Html.Editor` and `Html.EditorFor` have overlapping features with the Input Tag Helper. The Input Tag Helper will automatically set the `type` attribute; `Html.TextBox` and `Html.TextBoxFor` will not. `Html.Editor` and `Html.EditorFor` handle collections, complex objects and templates; the Input Tag Helper does not. The Input Tag Helper, `Html.EditorFor`  and  `Html.TextBoxFor` are strongly typed (they use lambda expressions); `Html.TextBox` and `Html.Editor` are not (they use expression names).

  ### Expression names

The `asp-for` attribute value is a [ModelExpression](https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Mvc/Rendering/ModelExpression/index.html) and the right hand side of a lambda expression. Therefore, `asp-for="Property1"` becomes `m => m.Property1` in the generated code which is why you don't need to prefix with `Model`. You can use the "@" character to start an inline expression and move before the `m.`:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML"} -->

````HTML

   @{
       var joe = "Joe";
   }
   <input asp-for="@joe" />
   ````

Generates the following:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML"} -->

````HTML

   <input type="text" id="joe" name="joe" value="Joe" />
   ````

  ### Navigating child properties

You can also navigate to child properties using the property path of the view model. Consider a more complex model class that contains a child `Address` property.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [1, 2, 3, 4], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/ViewModels/AddressViewModel.cs"} -->

````c#

    public class AddressViewModel
    {
        public string AddressLine1 { get; set; }
    }

   ````

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [8], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/ViewModels/RegisterAddressViewModel.cs"} -->

````c#

    public class RegisterAddressViewModel
    {
        public string Email { get; set; }

        [DataType(DataType.Password)]
        public string Password { get; set; }

        public AddressViewModel Address { get; set; }
    }


   ````

In the view, we bind to `Address.AddressLine1`:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [6], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/Views/Demo/RegisterAddress.cshtml"} -->

````HTML

   @model RegisterAddressViewModel

   <form asp-controller="Demo" asp-action="RegisterAddress" method="post">
       Email:  <input asp-for="Email" /> <br />
       Password: <input asp-for="Password" /><br />
       Address: <input asp-for="Address.AddressLine1" /><br />
       <button type="submit">Register</button>
   </form>
   ````

The following HTML is generated for `Address.AddressLine1`:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML"} -->

````HTML

   <input type="text" id="Address_AddressLine1" name="Address.AddressLine1" value="" />
   ````

  ### Expression names and Collections

Sample, a model containing an array of `Colors`:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [3], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/ViewModels/Person.cs"} -->

````c#

    public class Person
    {
        public List<string> Colors { get; set; }

        public int Age { get; set; }
    }

   ````

The action method:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#"} -->

````c#

   public IActionResult Edit(int id, int colorIndex)
   {
       ViewData["Index"] = colorIndex;
       return View(GetPerson(id));
   }
   ````

The following Razor shows how you access a specific `Color` element:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/Views/Demo/EditColor.cshtml"} -->

````HTML

   @model Person
   @{
       var index = (int)ViewData["index"];
   }

   <form asp-controller="ToDo" asp-action="Edit" method="post">
       @Html.EditorFor(m => m.Colors[index])
       <label asp-for="Age"></label>
       <input asp-for="Age" /><br />
       <button type="submit">Post</button>
   </form>
   ````

The *Views/Shared/EditorTemplates/String.cshtml* template:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/Views/Shared/EditorTemplates/String.cshtml"} -->

````HTML

   @model string

   <label asp-for="@Model"></label>
   <input asp-for="@Model" /> <br />

   ````

Sample using `List<T>`:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/ViewModels/ToDoItem.cs"} -->

````c#

    public class ToDoItem
    {
        public string Name { get; set; }

        public bool IsDone { get; set; }

   ````

The following Razor shows how to iterate over a collection:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "none", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/Views/Demo/Edit.cshtml"} -->

````none

   @model List<ToDoItem>

   <form asp-controller="ToDo" asp-action="Edit" method="post">
       <table>
           <tr> <th>Name</th> <th>Is Done</th> </tr>

           @for (int i = 0; i < Model.Count; i++)
           {
               <tr>
                   @Html.EditorFor(model => model[i])
               </tr>
           }

       </table>
       <button type="submit">Save</button>
   </form>

   ````

The *Views/Shared/EditorTemplates/ToDoItem.cshtml* template:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/Views/Shared/EditorTemplates/ToDoItem.cshtml"} -->

````HTML

   @model ToDoItem

   <td>
       <label asp-for="@Model.Name"></label>
       @Html.DisplayFor(model => model.Name)
   </td>
   <td>
       <input asp-for="@Model.IsDone" />
   </td>

   @*
       This template replaces the following Razor which evaluates the indexer three times.
       <td>
            <label asp-for="@Model[i].Name"></label>
            @Html.DisplayFor(model => model[i].Name)
        </td>
        <td>
            <input asp-for="@Model[i].IsDone" />
        </td>
   *@
   ````

Note:
   Always use `for` (and *not* `foreach`) to iterate over a list. Evaluating an indexer in a LINQ expression can be expensive and should be minimized.

Note:
   The commented sample code above shows how you would replace the lambda expression with the `@` operator to access each `ToDoItem` in the list.

  ## The Textarea Tag Helper

The [Textarea Tag Helper](https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Mvc/TagHelpers/TextAreaTagHelper/index.html) tag helper is  similar to the Input Tag Helper.

* Generates the `id` and `name` attributes, and the data validation attributes from the model for a [<textarea>](http://www.w3.org/wiki/HTML/Elements/textarea) element.

* Provides strong typing.

* HTML Helper alternative: `Html.TextAreaFor`

Sample:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/ViewModels/DescriptionViewModel.cs"} -->

````c#

   using System.ComponentModel.DataAnnotations;

   namespace FormsTagHelper.ViewModels
   {
       public class DescriptionViewModel
       {
           [MinLength(5)]
           [MaxLength(1024)]
           public string Description { get; set; }
       }
   }
   ````

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [4], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/Views/Demo/RegisterTextArea.cshtml"} -->

````HTML

   @model DescriptionViewModel

   <form asp-controller="Demo" asp-action="RegisterTextArea" method="post">
       <textarea asp-for="Description"></textarea>
       <button type="submit">Test</button>
   </form>
   ````

The following HTML is generated:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [2, 3, 4, 5, 6, 7, 8]}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML"} -->

````HTML

   <form method="post" action="/Demo/RegisterTextArea">
     <textarea data-val="true"
      data-val-maxlength="The field Description must be a string or array type with a maximum length of &#x27;1024&#x27;."
      data-val-maxlength-max="1024"
      data-val-minlength="The field Description must be a string or array type with a minimum length of &#x27;5&#x27;."
      data-val-minlength-min="5"
      id="Description" name="Description">
     </textarea>
     <button type="submit">Test</button>
     <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>" />
   </form>
   ````

  ## The Label Tag Helper

* Generates the label caption and `for` attribute on a [<label>](https://www.w3.org/wiki/HTML/Elements/label) element for an expression name

* HTML Helper alternative: `Html.LabelFor`.

The [Label Tag Helper](https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Mvc/TagHelpers/LabelTagHelper/index.html)  provides the following benefits over a pure HTML label element:

* You automatically get the descriptive label value from the `Display` attribute. The intended display name might change over time, and the combination of `Display` attribute and Label Tag Helper will apply the `Display` everywhere it's used.

* Less markup in source code

* Strong typing with the model property.

Sample:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/ViewModels/SimpleViewModel.cs"} -->

````c#

   using System.ComponentModel.DataAnnotations;

   namespace FormsTagHelper.ViewModels
   {
       public class SimpleViewModel
       {
           [Required]
           [EmailAddress]
           [Display(Name = "Email Address")]
           public string Email { get; set; }
       }
   }


   ````

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [4], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/Views/Demo/RegisterLabel.cshtml"} -->

````HTML

   @model SimpleViewModel

   <form asp-controller="Demo" asp-action="RegisterLabel" method="post">
       <label asp-for="Email"></label>
       <input asp-for="Email" /> <br />
   </form>
   ````

The following HTML is generated for the `<label>` element:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML"} -->

````HTML

   <label for="Email">Email Address</label>
   ````

The Label Tag Helper generated the `for` attribute value of "Email", which is the ID associated with the `<input>` element. The Tag Helpers generate consistent `id` and `for` elements so they can be correctly associated. The caption in this sample comes from the `Display` attribute. If the model didn't contain a `Display` attribute, the caption would be the property name of the expression.

  ## The Validation Tag Helpers

There are two Validation Tag Helpers. The [Validation Message Tag Helper](https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Mvc/TagHelpers/ValidationMessageTagHelper/index.html) (which displays a validation message for a single property on your model), and the [Validation Summary Tag Helper](https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Mvc/TagHelpers/ValidationSummaryTagHelper/index.html) (which displays a summary of validation errors). The [Input Tag Helper](https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Mvc/TagHelpers/InputTagHelper/index.html) adds HTML5 client side validation attributes to input elements based on data annotation attributes on your model classes. Validation is also performed on the server. The Validation Tag Helper displays these error messages when a validation error occurs.

  ### The Validation Message Tag Helper

* Adds the [HTML5](https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/HTML5)  `data-valmsg-for="property"` attribute to the [span](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/span) element, which attaches the validation error messages on the input field of the specified model property. When a client side validation error occurs, [jQuery](https://jquery.com/) displays the error message in the `<span>` element.

* Validation also takes place on the server. Clients may have JavaScript disabled and some validation can only be done on the server side.

* HTML Helper alternative: `Html.ValidationMessageFor`

The [Validation Message Tag Helper](https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Mvc/TagHelpers/ValidationMessageTagHelper/index.html)  is used with the `asp-validation-for` attribute on a HTML [span](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/span) element.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML"} -->

````HTML

   <span asp-validation-for="Email"></span>
   ````

The Validation Message Tag Helper will generate the following HTML:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML"} -->

````HTML

   <span class="field-validation-valid"
     data-valmsg-for="Email"
     data-valmsg-replace="true"></span>
   ````

You generally use the [Validation Message Tag Helper](https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Mvc/TagHelpers/ValidationMessageTagHelper/index.html)  after an `Input` Tag Helper for the same property. Doing so displays any validation error messages near the input that caused the error.

Note:
   You must have a view with the correct JavaScript and [jQuery](https://jquery.com/) script references in place for client side validation. See [Model Validation](../models/validation.md) for more information.

When a server side validation error occurs (for example when you have custom server side validation or client-side validation is disabled), MVC places that error message as the body of the `<span>` element.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML"} -->

````HTML

   <span class="field-validation-error" data-valmsg-for="Email"
               data-valmsg-replace="true">
      The Email Address field is required.
   </span>
   ````

  ### The Validation Summary Tag Helper

* Targets `<div>` elements with the `asp-validation-summary` attribute

* HTML Helper alternative: `@Html.ValidationSummary`

The [Validation Summary Tag Helper](https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Mvc/TagHelpers/ValidationSummaryTagHelper/index.html)  is used to display a summary of validation messages. The `asp-validation-summary` attribute value can be any of the following:

<!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- Skip node --><!-- table -->

  ### Sample

In the following example, the data model is decorated with `DataAnnotation` attributes, which generates validation error messages on the `<input>` element.  When a validation error occurs, the Validation Tag Helper displays the error message:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/ViewModels/RegisterViewModel.cs"} -->

````c#

   using System.ComponentModel.DataAnnotations;

   namespace FormsTagHelper.ViewModels
   {
       public class RegisterViewModel
       {
           [Required]
           [EmailAddress]
           [Display(Name = "Email Address")]
           public string Email { get; set; }

           [Required]
           [DataType(DataType.Password)]
           public string Password { get; set; }
       }
   }
   ````

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [4, 6, 8], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/Views/Demo/RegisterValidation.cshtml"} -->

````HTML

   @model RegisterViewModel

   <form asp-controller="Demo" asp-action="RegisterValidation" method="post">
       <div asp-validation-summary="ValidationSummary.ModelOnly"></div>
       Email:  <input asp-for="Email" /> <br />
       <span asp-validation-for="Email"></span><br />
       Password: <input asp-for="Password" /><br />
       <span asp-validation-for="Password"></span><br />
       <button type="submit">Register</button>
   </form>

   ````

The generated HTML (when the model is valid):

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [2, 3, 8, 9, 12, 13]}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML"} -->

````HTML

   <form action="/DemoReg/Register" method="post">
     <div class="validation-summary-valid" data-valmsg-summary="true">
     <ul><li style="display:none"></li></ul></div>
     Email:  <input name="Email" id="Email" type="email" value=""
      data-val-required="The Email field is required."
      data-val-email="The Email field is not a valid e-mail address."
      data-val="true"> <br>
     <span class="field-validation-valid" data-valmsg-replace="true"
      data-valmsg-for="Email"></span><br>
     Password: <input name="Password" id="Password" type="password"
      data-val-required="The Password field is required." data-val="true"><br>
     <span class="field-validation-valid" data-valmsg-replace="true"
      data-valmsg-for="Password"></span><br>
     <button type="submit">Register</button>
     <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>" />
   </form>
   ````

  ## The Select Tag Helper

* Generates [select](https://www.w3.org/wiki/HTML/Elements/select) and associated [option](https://www.w3.org/wiki/HTML/Elements/option) elements for properties of your model.

* Has an HTML Helper alternative `Html.DropDownListFor` and `Html.ListBoxFor`

The [Select Tag Helper](https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Mvc/TagHelpers/SelectTagHelper/index.html) `asp-for` specifies the model property  name for the [select](https://www.w3.org/wiki/HTML/Elements/select) element  and `asp-items` specifies the [option](https://www.w3.org/wiki/HTML/Elements/option) elements.  For example:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/Views/Home/Index.cshtml"} -->

````HTML

    <select asp-for="Country" asp-items="Model.Countries"></select> 

   ````

Sample:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/ViewModels/CountryViewModel.cs"} -->

````c#

   using Microsoft.AspNetCore.Mvc.Rendering;
   using System.Collections.Generic;

   namespace FormsTagHelper.ViewModels
   {
       public class CountryViewModel
       {
           public string Country { get; set; }

           public List<SelectListItem> Countries { get; } = new List<SelectListItem>
           {
               new SelectListItem { Value = "MX", Text = "Mexico" },
               new SelectListItem { Value = "CA", Text = "Canada" },
               new SelectListItem { Value = "US", Text = "USA"  },
           };
       }
   }
   ````

The `Index` method initializes the `CountryViewModel`, sets the selected country and passes it to the `Index` view.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/Controllers/HomeController.cs"} -->

````c#

     public IActionResult Index()
     {
         var model = new CountryViewModel();
         model.Country = "CA";
         return View(model);
     }

   ````

The HTTP POST `Index` method displays the selection:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/Controllers/HomeController.cs"} -->

````c#

     [HttpPost]
     [ValidateAntiForgeryToken]
     public IActionResult Index(CountryViewModel model)
     {
         if (ModelState.IsValid)
         {
             var msg = model.Country +  " selected";
             return RedirectToAction("IndexSuccess", new { message = msg});
         }

         // If we got this far, something failed; redisplay form.
         return View(model);
     }

   ````

The `Index` view:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [4], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/Views/Home/Index.cshtml"} -->

````HTML

   @model CountryViewModel

   <form asp-controller="Home" asp-action="Index" method="post">
       <select asp-for="Country" asp-items="Model.Countries"></select> 
       <br /><button type="submit">Register</button>
   </form>

   ````

Which generates the following HTML (with "CA" selected):

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [2, 3, 4, 5, 6]}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML"} -->

````HTML

   <form method="post" action="/">
     <select id="Country" name="Country">
       <option value="MX">Mexico</option>
       <option selected="selected" value="CA">Canada</option>
       <option value="US">USA</option>
     </select>
       <br /><button type="submit">Register</button>
     <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>" />
   </form>
   ````

Note:
   We do not recommend using `ViewBag` or `ViewData` with the Select Tag Helper. A view model is more robust at providing MVC metadata and generally less problematic.

The `asp-for` attribute value is a special case and doesn't require a `Model` prefix, the other Tag Helper attributes do (such as `asp-items`)

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/Views/Home/Index.cshtml"} -->

````HTML

    <select asp-for="Country" asp-items="Model.Countries"></select> 

   ````

  ### Enum binding

It's often convenient to use `<select>` with an `enum` property and generate the [SelectListItem](https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Mvc/Rendering/SelectListItem/index.html?highlight=selectlistitem) elements from the `enum` values.

Sample:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/ViewModels/CountryEnumViewModel.cs"} -->

````c#

    public class CountryEnumViewModel
    {
        public CountryEnum EnumCountry { get; set; }
    }

   ````

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/ViewModels/CountryEnum.cs"} -->

````c#

   using System.ComponentModel.DataAnnotations;

   namespace FormsTagHelper.ViewModels
   {
       {
           Mexico,
           [Display(Name = "United States of America")]
           USA,
           Canada,
           France,
           Germany,
           Spain
       }
   }
   ````

The [GetEnumSelectList](https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Mvc/Rendering/IHtmlHelper/index.html) method generates a [SelectList](https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Mvc/Rendering/SelectList/index.html) object for an enum.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [5], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/Views/Home/IndexEnum.cshtml"} -->

````HTML

   @model CountryEnumViewModel

   <form asp-controller="Home" asp-action="IndexEnum" method="post">
       <select asp-for="EnumCountry" 
               asp-items="Html.GetEnumSelectList<CountryEnum>()"> >
       </select> 
       <br /><button type="submit">Register</button>
   </form>

   ````

You can decorate your enumerator list with the `Display` attribute to get a richer UI:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [5, 7], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/ViewModels/CountryEnum.cs"} -->

````c#

   using System.ComponentModel.DataAnnotations;

   namespace FormsTagHelper.ViewModels
   {
       public enum CountryEnum
       {
           [Display(Name = "United Mexican States")]
           Mexico,
           [Display(Name = "United States of America")]
           USA,
           Canada,
           France,
           Germany,
           Spain
       }
   }
   ````

The following HTML is generated:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [4, 5]}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML"} -->

````HTML

     <form method="post" action="/Home/IndexEnum">
         <select data-val="true" data-val-required="The EnumCountry field is required."
                 id="EnumCountry" name="EnumCountry">
             <option value="0">United Mexican States</option>
             <option value="1">United States of America</option>
             <option value="2">Canada</option>
             <option value="3">France</option>
             <option value="4">Germany</option>
             <option selected="selected" value="5">Spain</option>
         </select>
         <br /><button type="submit">Register</button>
         <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>" />
    </form>
   ````

  ### Option Group

The HTML  [<optgroup>](https://www.w3.org/wiki/HTML/Elements/optgroup) element is generated when the view model contains one or more [SelectListGroup](https://docs.asp.net/projects/api/en/latest/autoapi/Microsoft/AspNetCore/Mvc/Rendering/SelectListGroup/index.html) objects.

The `CountryViewModelGroup` groups the `SelectListItem` elements into the "North America" and "Europe" groups:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [5, 6, 14, 20, 26, 32, 38, 44], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/ViewModels/CountryViewModelGroup.cs"} -->

````c#

    public class CountryViewModelGroup
    {
        public CountryViewModelGroup()
        {
            var NorthAmericaGroup = new SelectListGroup { Name = "North America" };
            var EuropeGroup = new SelectListGroup { Name = "Europe" };

            Countries = new List<SelectListItem>
            {
                new SelectListItem
                {
                    Value = "MEX",
                    Text = "Mexico",
                    Group = NorthAmericaGroup
                },
                new SelectListItem
                {
                    Value = "CAN",
                    Text = "Canada",
                    Group = NorthAmericaGroup
                },
                new SelectListItem
                {
                    Value = "US",
                    Text = "USA",
                    Group = NorthAmericaGroup
                },
                new SelectListItem
                {
                    Value = "FR",
                    Text = "France",
                    Group = EuropeGroup
                },
                new SelectListItem
                {
                    Value = "ES",
                    Text = "Spain",
                    Group = EuropeGroup
                },
                new SelectListItem
                {
                    Value = "DE",
                    Text = "Germany",
                    Group = EuropeGroup
                }
          };
        }

        public string Country { get; set; }

        public List<SelectListItem> Countries { get; }
    }

   ````

The two groups are shown below:

![image](working-with-forms/_static/grp.png)

The generated HTML:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [3, 4, 5, 6, 7, 8, 9, 10, 11, 12]}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML"} -->

````HTML

     <form method="post" action="/Home/IndexGroup">
         <select id="Country" name="Country">
             <optgroup label="North America">
                 <option value="MEX">Mexico</option>
                 <option value="CAN">Canada</option>
                 <option value="US">USA</option>
             </optgroup>
             <optgroup label="Europe">
                 <option value="FR">France</option>
                 <option value="ES">Spain</option>
                 <option value="DE">Germany</option>
             </optgroup>
         </select>
         <br /><button type="submit">Register</button>
         <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>" />
    </form>
   ````

  ### Multiple select

The Select Tag Helper  will automatically generate the [multiple = "multiple"](https://www.w3.org/TR/html-markup/select.html#select.attrs.multiple)  attribute if the property specified in the `asp-for` attribute is an `IEnumerable`. For example, given the following model:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [6], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/ViewModels/CountryViewModelIEnumerable.cs"} -->

````c#

   using Microsoft.AspNetCore.Mvc.Rendering;
   using System.Collections.Generic;

   namespace FormsTagHelper.ViewModels
   {
       public class CountryViewModelIEnumerable
       {
           public IEnumerable<string> CountryCodes { get; set; }

           public List<SelectListItem> Countries { get; } = new List<SelectListItem>
           {
               new SelectListItem { Value = "MX", Text = "Mexico" },
               new SelectListItem { Value = "CA", Text = "Canada" },
               new SelectListItem { Value = "US", Text = "USA"    },
               new SelectListItem { Value = "FR", Text = "France" },
               new SelectListItem { Value = "ES", Text = "Spain"  },
               new SelectListItem { Value = "DE", Text = "Germany"}
            };
       }
   }

   ````

With the following view:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [4], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/Views/Home/IndexMultiSelect.cshtml"} -->

````HTML

   @model CountryViewModelIEnumerable

   <form asp-controller="Home" asp-action="IndexMultiSelect" method="post">
       <select asp-for="CountryCodes" asp-items="Model.Countries"></select> 
       <br /><button type="submit">Register</button>
   </form>

   ````

Generates the following HTML:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [3]}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML"} -->

````HTML

   <form method="post" action="/Home/IndexMultiSelect">
       <select id="CountryCodes"
       multiple="multiple"
       name="CountryCodes"><option value="MX">Mexico</option>
   <option value="CA">Canada</option>
   <option value="US">USA</option>
   <option value="FR">France</option>
   <option value="ES">Spain</option>
   <option value="DE">Germany</option>
   </select>
       <br /><button type="submit">Register</button>
     <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>" />
   </form>
   ````

  ### No selection

To allow for no selection, add a "not specified" option to the select list. If the property is a [value type](https://msdn.microsoft.com/en-us/library/s1ax56ch.aspx), you'll have to make it [nullable](https://msdn.microsoft.com/en-us/library/2cf62fcy.aspx).

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [5], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/Views/Home/IndexEmpty.cshtml"} -->

````HTML

   @model CountryViewModel

   <form asp-controller="Home" asp-action="IndexEmpty" method="post">
       <select asp-for="Country" asp-items="Model.Countries">
           <option value="">&lt;none&gt;</option>
       </select> 
       <br /><button type="submit">Register</button>
   </form>

   ````

If you find yourself using the "not specified" option in multiple pages, you can create a template to eliminate repeating the HTML:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [5], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/Views/Home/IndexEmptyTemplate.cshtml"} -->

````HTML

   @model CountryViewModel

   <form asp-controller="Home" asp-action="IndexEmpty" method="post">
       @Html.EditorForModel()
       <br /><button type="submit">Register</button>
   </form>

   ````

The *Views/Shared/EditorTemplates/CountryViewModel.cshtml* template:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/Views/Shared/EditorTemplates/CountryViewModel.cshtml"} -->

````HTML

   @model CountryViewModel

   <select asp-for="Country" asp-items="Model.Countries">
       <option value="">--none--</option>
   </select>


   ````

Adding HTML [<option>](https://www.w3.org/wiki/HTML/Elements/option) elements is not limited to the *No selection* case. For example, the following view and action method will generate HTML similar to the code above:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/Controllers/HomeController.cs"} -->

````c#

     public IActionResult IndexOption(int id)
     {
         var model = new CountryViewModel();
         model.Country = "CA";
         return View(model);
     }

   ````

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/views/working-with-forms/sample/final/Views/Home/IndexOption.cshtml"} -->

````HTML

   @model CountryViewModel

   <form asp-controller="Home" asp-action="IndexEmpty" method="post">
       <select asp-for="Country">
           <option value="">&lt;none&gt;</option>
           <option value="MX">Mexico</option>
           <option value="CA">Canada</option>
           <option value="US">USA</option>
       </select> 
       <br /><button type="submit">Register</button>
   </form>

   ````

The correct `<option>` element will be selected ( contain the `selected="selected"` attribute) depending on the current `Country` value.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [5]}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "HTML"} -->

````HTML

    <form method="post" action="/Home/IndexEmpty">
        <select id="Country" name="Country">
            <option value="">&lt;none&gt;</option>
            <option value="MX">Mexico</option>
            <option value="CA" selected="selected">Canada</option>
            <option value="US">USA</option>
        </select>
        <br /><button type="submit">Register</button>
     <input name="__RequestVerificationToken" type="hidden" value="<removed for brevity>" />
   </form>
   ````

  ## Additional Resources

* [Tag Helpers](tag-helpers/intro.md)

* [HTML Form element](https://www.w3.org/TR/html401/interact/forms.html)

* [Request Verification Token](http://www.asp.net/mvc/overview/security/xsrfcsrf-prevention-in-aspnet-mvc-and-web-pages)

* [Model Binding](../models/model-binding.md)

* [Model Validation](../models/validation.md)

* [data annotations](https://msdn.microsoft.com/en-us/library/system.componentmodel.dataannotations.aspx)

* [Code snippets for this document](https://github.com/aspnet/Docs/tree/master/aspnet/mvc/views/forms/sample).
