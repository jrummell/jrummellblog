---
permalink: /xval-with-webforms
redirect_from: /xval-with-webforms/
title: xVal with WebForms 
layout: post
date: 2009-07-15 22:32:00
published: true
tags: [validation, xval-webforms]
---


**Update**: See [xVal with WebForms Part 2](/xval-with-webforms-part-2) for a better implementation.


## What is xVal and why would anyone want to use it?

> xVal is a validation framework for ASP.NET MVC applications. It makes it easy to link up your choice of server-side validation mechanism with your choice of client-side validation library, neatly fitting both into ASP.NET MVC architecture and conventions.

![](http://blog.codeville.net/wp-content/uploads/2009/01/image-thumb.png)

See the [CodePlex](http://xval.codeplex.com/) page for more information.

Basically, what xVal does, is take your validation rules and perform server and client side validation based on those rules. That means you don’t have to duplicate model validation at the page level. Now you’re probably thinking, “Isn’t it for MVC?”. It is. But I, and at least [two others](http://xval.codeplex.com/Thread/View.aspx?ThreadId=60906), would like to take advantage of xVal’s features in traditional ASP.NET WebForm projects.


## Getting it to work with WebForms

I finally found some time last night to see what it would take to get xVal working in an ASP.NET Web Application Project. After a few hours I had something. I only needed to add two classes on top of xVal, DataAnnotationsValidationRunner and ModelValidator.

    public static class DataAnnotationsValidationRunner
    {
        public static IEnumerable<ErrorInfo> GetErrors(object instance, string propertyName)
        {
            return from prop in TypeDescriptor.GetProperties(instance).Cast<PropertyDescriptor>()
                from attribute in prop.Attributes.OfType<ValidationAttribute>()
                where prop.Name == propertyName && !attribute.IsValid(prop.GetValue(instance))
                select new ErrorInfo(prop.Name, attribute.FormatErrorMessage(string.Empty), instance);
        }
    }

This is based on the implementation in the [xVal demo](http://blog.codeville.net/2009/01/10/xval-a-validation-framework-for-aspnet-mvc/). I added a second parameter to GetErrors() that allows the runner to check a specific property.

    public class ModelValidator : BaseValidator
    {
        private ValidationInfo _validationInfo;
    
        public string ModelType
        {
            get { return (string) ViewState["ModelType"]; }
            set { ViewState["ModelType"] = value; }
        }
    
        public string ModelProperty
        {
            get { return (string) ViewState["ModelProperty"]; }
            set { ViewState["ModelProperty"] = value; }
        }
    
        protected override bool EvaluateIsValid()
        {
            Type type = Type.GetType(ModelType);
    
            object model = Activator.CreateInstance(type);
    
            IEnumerable<ErrorInfo> errors = DataAnnotationsValidationRunner.GetErrors(model, ModelProperty);
    
            StringBuilder errorBuilder = new StringBuilder();
            foreach (ErrorInfo error in errors)
            {
                errorBuilder.AppendLine(error.ErrorMessage);
            }
    
            ErrorMessage = errorBuilder.ToString();
    
            return ErrorMessage.Length > 0;
        }
    
        protected override void Render(HtmlTextWriter writer)
        {
            Type type = Type.GetType(ModelType);
            _validationInfo = new ValidationInfo(ActiveRuleProviders.GetRulesForType(type), String.Empty);
    
            writer.Write(_validationInfo.ToString());
        }
    }

This is the ASP.NET WebForms version of `<%= Html.ClientSideValidation<Booking>(“booking”) %>`, an implementation of [BaseValidator](http://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.basevalidator.aspx). It uses [ValidationInfo](http://xval.codeplex.com/sourcecontrol/changeset/view/21650?projectName=xval#260910) for rendering the client validation script and DataAnnotationsValidationRunner for the server side validation.

You can use it like this:

    public class Customer
    {
        [Required, StringLength(20)]
        public string Name { get; set; }
    }

    <script type="text/javascript" src="xVal.AspNetNative.js"></script>
    <div>
        <asp:Label ID="lblName" runat="server" AssociatedControlID="Name">Customer Name:</asp:Label>
        <asp:TextBox ID="Name" runat="server" />
        <val:ModelValidator ID="validator" runat="server" ModelType="xVal.WebForms.Test.Customer, xVal.WebForms.Test"
            ModelProperty="Name" ControlToValidate="Name" />
    </div>
    <asp:Button ID="btnSubmit" runat="server" Text="Submit" />

There are a few things to note here:

- The TextBox ID must be the same as the model’s property name. This is because the xVal javascript is expecting the control’s ID to be PropertyName or prefix.PropertyName (MVC naming conventions). This could probably be fixed by modifying the [client](http://xval.codeplex.com/sourcecontrol/changeset/view/21650?projectName=xval#279841) [side](http://xval.codeplex.com/sourcecontrol/changeset/view/21650?projectName=xval#279846) plugins.
- The ControlToValidate property on ModelValidator doesn’t do anything, but it’s required by any implementation of BaseValidator (it will throw an exception if you omit it). This could probably be avoided by having ModelValidator inherit from [Control](http://msdn.microsoft.com/en-us/library/system.web.ui.control.aspx) and implement [IValidator](http://msdn.microsoft.com/en-us/library/system.web.ui.ivalidator.aspx) instead.
- xVal depends on System.Web.Mvc. It uses the TagBuilder and ModelState classes in a [few](http://xval.codeplex.com/sourcecontrol/changeset/view/21650?projectName=xval#260910) [places](http://xval.codeplex.com/sourcecontrol/changeset/view/21650?projectName=xval#72733). There’s really now way around this without refactoring the MVC specific stuff into a separate assembly.

I’ll post an update when I have a chance to work on the first two items.
