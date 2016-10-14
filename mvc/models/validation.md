---
uid: mvc/models/validation
---
  # Model Validation

By [Rachel Appel](http://github.com/rachelappel)

In this article:

  ## Introduction to model validation

Before an app stores data in a database, the app must validate the data. Data must be checked for potential security threats, verified that it is appropriately formatted by type and size, and it must conform to your rules. Validation is necessary although it can be redundant and tedious to implement. In MVC, validation happens on both the client and server.

Fortunately, .NET has abstracted validation into validation attributes. These attributes contain validation code, thereby reducing the amount of code you must write.

  ## Validation Attributes

Validation attributes are a way to configure model validation so it's similar conceptually to validation on fields in database tables. This includes constraints such as assigning data types or required fields. Other types of validation include applying patterns to data to enforce business rules, such as a credit card, phone number, or email address. Validation attributes make enforcing these requirements much simpler and easier to use.

Below is an annotated `Movie` model from an app that stores information about movies and TV shows. Most of the properties are required and several string properties have length requirements. Additionally, there is a numeric range restriction in place for the `Price` property from 0 to $999.99, along with a custom validation attribute.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/models/validation/sample/Movie.cs"} -->

````c#

   public class Movie
   {
       public int Id { get; set; }

       [Required]
       [StringLength(100)]
       public string Title { get; set; }

       [Required]
       [ClassicMovie(1960)]
       [DataType(DataType.Date)]
       public DateTime ReleaseDate { get; set; }

       [Required]
       [StringLength(1000)]
       public string Description { get; set; }

       [Required]
       [Range(0, 999.99)]
       public decimal Price { get; set; }

       [Required]
       public Genre Genre { get; set; }

       public bool Preorder { get; set; }
   }

   ````

Simply reading through the model reveals the rules about data for this app, making it easier to maintain the code. Below are several popular built-in validation attributes:

* `[CreditCard]`: Validates the property has a credit card format.

* `[Compare]`: Validates two properties in a model match.

* `[EmailAddress]`: Validates the property has an email format.

* `[Phone]`: Validates the property has a telephone format.

* `[Range]`: Validates the property value falls within the given range.

* `[RegularExpression]`: Validates that the data matches the specified regular expression.

* `[Required]`: Makes a property required.

* `[StringLength]`: Validates that a string property has at most the given maximum length.

* `[Url]`: Validates the property has a URL format.

MVC supports any attribute that derives from `ValidationAttribute` for validation purposes. Many useful validation attributes can be found in the [System.ComponentModel.DataAnnotations](https://msdn.microsoft.com/en-us/library/system.componentmodel.dataannotations(v=vs.110).aspx) namespace.

There may be instances where you need more features than built-in attributes provide. For those times, you can create custom validation attributes by deriving from `ValidationAttribute` or changing your model to implement `IValidatableObject`.

  ## Model State

Model state represents validation errors in submitted HTML form values.

MVC will continue validating fields until reaches the maximum number of errors (200 by default). You can configure this number by inserting the following code into the `ConfigureServices` method in the `Startup.cs` file:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/models/validation/sample/Startup.cs"} -->

````c#

   services.AddMvc(options => options.MaxModelValidationErrors = 50);

   ````

  ## Handling Model State Errors

Model validation occurs prior to each controller action being invoked, and it is the action method’s responsibility to inspect `ModelState.IsValid` and react appropriately. In many cases, the appropriate reaction is to return some kind of error response, ideally detailing the reason why model validation failed.

Some apps will choose to follow a standard convention for dealing with model validation errors, in which case a filter may be an appropriate place to implement such a policy. You should test how your actions behave with valid and invalid model states.

  ## Manual validation

After model binding and validation are complete, you may want to repeat parts of it. For example, a user may have entered text in a field expecting an integer, or you may need to compute a value for a model's property.

You may need to run validation manually. To do so, call the `TryValidateModel` method, as shown here:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/models/validation/sample/MoviesController.cs"} -->

````c#

   TryValidateModel(movie);

   ````

  ## Custom validation

Validation attributes work for most validation needs. However, some validation rules are specific to your business, as they're not just generic data validation such as ensuring a field is required or that it conforms to a range of values. For these scenarios, custom validation attributes are a great solution. Creating your own custom validation attributes in MVC is easy. Just inherit from the `ValidationAttribute`, and override the `IsValid` method. The `IsValid` method accepts two parameters, the first is an object named *value* and the second is a `ValidationContext` object named *validationContext*. *Value* refers to the actual value from the field that your custom validator is validating.

In the following sample, a business rule states that users may not set the genre to *Classic* for a movie released after 1960. The `[ClassicMovie]` attribute checks the genre first, and if it is a classic, then it checks the release date to see that it is later than 1960. If it is released after 1960, validation fails. The attribute accepts an integer parameter representing the year that you can use to validate data. You can capture the value of the parameter in the attribute's constructor, as shown here:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/models/validation/sample/ClassicMovieAttribute.cs"} -->

````c#

   public class ClassicMovieAttribute : ValidationAttribute, IClientModelValidator
   {
       private int _year;

       public ClassicMovieAttribute(int Year)
       {
           _year = Year;
       }

       protected override ValidationResult IsValid(object value, ValidationContext validationContext)
       {
           Movie movie = (Movie)validationContext.ObjectInstance;

           if (movie.Genre == Genre.Classic && movie.ReleaseDate.Year > _year)
           {
               return new ValidationResult(GetErrorMessage());
           }

           return ValidationResult.Success;
       }

   ````

The `movie` variable above represents a `Movie` object that contains the data from the form submission to validate. In this case, the validation code checks the date and genre in the `IsValid` method of the `ClassicMovieAttribute` class as per the rules. Upon successful validation `IsValid` returns a `ValidationResult.Success` code, and when validation fails, a `ValidationResult` with an error message. When a user modifies the `Genre` field and submits the form, the `IsValid` method of the `ClassicMovieAttribute` will verify whether the movie is a classic. Like any built-in attribute, apply the `ClassicMovieAttribute` to a property such as `ReleaseDate` to ensure validation happens, as shown in the previous code sample. Since the example works only with `Movie` types, a better option is to use `IValidatableObject` as shown in the following paragraph.

Alternatively, this same code could be placed in the model by implementing the `Validate` method on the `IValidatableObject` interface. While custom validation attributes work well for validating individual properties, implementing `IValidatableObject` can be used to implement class-level validation as seen here.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/models/validation/sample/MovieIValidatable.cs"} -->

````c#

   public IEnumerable<ValidationResult> Validate(ValidationContext validationContext)
   {
       if (Genre == Genre.Classic && ReleaseDate.Year > _classicYear)
       {
           yield return new ValidationResult(
               "Classic movies must have a release year earlier than " + _classicYear,
               new[] { "ReleaseDate" });
       }
   }

   ````

  ## Client side validation

Client side validation is a great convenience for users. It saves time they would otherwise spend waiting for a round trip to the server. In business terms, even a few fractions of seconds multiplied hundreds of times each day adds up to be a lot of time, expense, and frustration. Straightforward and immediate validation enables users to work more efficiently and produce better quality input and output.

You must have a view with the proper JavaScript script references in place for client side validation to work as you see here.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "html", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/models/validation/sample/Views/Shared/_Layout.cshtml"} -->

````html

   <script src="https://ajax.aspnetcdn.com/ajax/jQuery/jquery-1.11.3.min.js"></script>

   ````

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "html", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/models/validation/sample/Views/Shared/_ValidationScriptsPartial.cshtml"} -->

````html

   <script src="https://ajax.aspnetcdn.com/ajax/jquery.validate/1.14.0/jquery.validate.min.js"></script>
   <script src="https://ajax.aspnetcdn.com/ajax/jquery.validation.unobtrusive/3.2.6/jquery.validate.unobtrusive.min.js"></script>
   ````

MVC uses validation attributes in addition to type metadata from model properties to validate data and display any error messages using JavaScript. When you use MVC to render form elements from a model using [Tag Helpers](https://docs.asp.net/en/latest/mvc/views/tag-helpers/index.html) or [HTML helpers](https://docs.asp.net/en/latest/mvc/views/html-helpers.html) it will add HTML 5 [data- attributes](http://w3c.github.io/html/dom.html#embedding-custom-non-visible-data-with-the-data-attributes) in the form elements that need validation, as shown below. MVC generates the `data-` attributes for both built-in and custom attributes. You can display validation errors on the client using the relevant tag helpers as shown here:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [4, 5], "linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "html", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/models/validation/sample/Views/Movies/Create.cshtml"} -->

````html

   <div class="form-group">
       <label asp-for="ReleaseDate" class="col-md-2 control-label"></label>
       <div class="col-md-10">
           <input asp-for="ReleaseDate" class="form-control" />
           <span asp-validation-for="ReleaseDate" class="text-danger"></span>
       </div>
   </div>

   ````

The tag helpers above render the HTML below. Notice that the `data-` attributes in the HTML output correspond to the validation attributes for the `ReleaseDate` property. The `data-val-required` attribute below contains an error message to display if the user doesn't fill in the release date field, and that message displays in the accompanying `<span>` element.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"hl_lines": [8, 9, 10, 11, 12]}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "html"} -->

````html

   <form action="/movies/Create" method="post">
     <div class="form-horizontal">
       <h4>Movie</h4>
       <div class="text-danger"></div>
       <div class="form-group">
         <label class="col-md-2 control-label" for="ReleaseDate">ReleaseDate</label>
         <div class="col-md-10">
           <input class="form-control" type="datetime"
           data-val="true" data-val-required="The ReleaseDate field is required."
           id="ReleaseDate" name="ReleaseDate" value="" />
           <span class="text-danger field-validation-valid"
           data-valmsg-for="ReleaseDate" data-valmsg-replace="true"></span>
         </div>
       </div>
       </div>
   </form>
   ````

Client-side validation prevents submission until the form is valid. The Submit button runs JavaScript that either submits the form or displays error messages.

MVC determines type attribute values based on the .NET data type of a property, possibly overridden using `[DataType]` attributes. The base `[DataType]` attribute does no real server-side validation. Browsers choose their own error messages and display those errors however they wish, however the jQuery Validation Unobtrusive package can override the messages and display them consistently with others. This happens most obviously when users apply `[DataType]` subclasses such as `[EmailAddress]`.

  ## IClientModelValidator

You may create client side logic for your custom attribute, and [unobtrusive validation](http://jqueryvalidation.org/documentation/) will execute it on the client for you automatically as part of validation. The first step is to control what data- attributes are added by implementing the `IClientModelValidator` interface as shown here:

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/models/validation/sample/ClassicMovieAttribute.cs"} -->

````c#

   public void AddValidation(ClientModelValidationContext context)
   {
       if (context == null)
       {
           throw new ArgumentNullException(nameof(context));
       }

       MergeAttribute(context.Attributes, "data-val", "true");
       MergeAttribute(context.Attributes, "data-val-classicmovie", GetErrorMessage());

       var year = _year.ToString(CultureInfo.InvariantCulture);
       MergeAttribute(context.Attributes, "data-val-classicmovie-year", year);
   }

   ````

Attributes that implement this interface can add HTML attributes to generated fields. Examining the output for the `ReleaseDate` element reveals HTML that is similar to the previous example, except now there is a `data-val-classicmovie` attribute that was defined in the `AddValidation` method of `IClientModelValidator`.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "html"} -->

````html

   <input class="form-control" type="datetime"
   data-val="true"
   data-val-classicmovie="Classic movies must have a release year earlier than 1960"
   data-val-classicmovie-year="1960"
   data-val-required="The ReleaseDate field is required."
   id="ReleaseDate" name="ReleaseDate" value="" />
   ````

Unobtrusive validation uses the data in the `data-` attributes to display error messages. However, jQuery doesn't know about rules or messages until you add them to jQuery's `validator` object. This is shown in the example below that adds a method named `classicmovie` containing custom client validation code to the jQuery `validator` object.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "javascript", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/models/validation/sample/Views/Movies/Create.cshtml"} -->

````javascript

   $(function () {
       jQuery.validator.addMethod('classicmovie',
           function (value, element, params) {
               // Get element value. Classic genre has value '0'.
               var genre = $(params[0]).val(),
                   year = params[1],
                   date = new Date(value);
               if (genre && genre.length > 0 && genre[0] === '0') {
                   // Since this is a classic movie, invalid if release date is after given year.
                   return date.getFullYear() <= year;
               }

               return true;
           });

       jQuery.validator.unobtrusive.adapters.add('classicmovie',
           [ 'element', 'year' ],
           function (options) {
               var element = $(options.form).find('select#Genre')[0];
               options.rules['classicmovie'] = [element, parseInt(options.params['year'])];
               options.messages['classicmovie'] = options.message;
           });
   }(jQuery));

   ````

Now jQuery has the information to execute the custom JavaScript validation as well as the error message to display if that validation code returns false.

  ## Remote validation

Remote validation is a great feature to use when you need to validate data on the client against data on the server. For example, your app may need to verify whether an email or user name is already in use, and it must query a large amount of data to do so. Downloading large sets of data for validating one or a few fields consumes too many resources. It may also expose sensitive information. An alternative is to make a round-trip request to validate a field.

You can implement remote validation in a two step process. First, you must annotate your model with the `[Remote]` attribute. The `[Remote]` attribute accepts multiple overloads you can use to direct client side JavaScript to the appropriate code to call. The example points to the `VerifyEmail` action method of the `Users` controller.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "c#", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/models/validation/sample/User.cs"} -->

````c#

   public class User
   {
       [Remote(action: "VerifyEmail", controller: "Users")]
       public string Email { get; set; }
   }

   ````

The second step is putting the validation code in the corresponding action method as defined in the `[Remote]` attribute. It returns a `JsonResult` that the client side can use to proceed or pause and display an error if needed.

<!-- literal_block {"ids": [], "names": [], "highlight_args": {"linenostart": 1}, "backrefs": [], "dupnames": [], "linenos": false, "classes": [], "xml:space": "preserve", "language": "none", "source": "/Users/shirhatti/src/Docs/aspnet/mvc/models/validation/sample/UsersController.cs"} -->

````none

   [AcceptVerbs("Get", "Post")]
   public IActionResult VerifyEmail(string email)
   {
       if (!_userRepository.VerifyEmail(email))
       {
           return Json(data: $"Email {email} is already in use.");
       }

       return Json(data: true);
   }

   ````

Now when users enter an email, JavaScript in the view makes a remote call to see if that email has been taken, and if so, then displays the error message. Otherwise, the user can submit the form as usual.
