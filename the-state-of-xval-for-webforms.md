---
permalink: /the-state-of-xval-for-webforms
title: The State of xVal for WebForms
date: 2011-03-06 16:19:05
published: true
tags: validation xval-webforms
---


I’ve been neglecting [xVal for WebForms](http://xvalwebforms.codeplex.com/) for a while now, mainly because I’m not sure which direction to take it. The [xVal](http://xval.codeplex.com/) project is now deprecated in favor of the client side validation support introduced in ASP.NET MVC 2. This is obviously a problem since xVal for WebForms is built on top of xVal.

I think there are a few directions the project could take. The more traditional WebForms approach would be to simply generate the standard System.Web.Web.UI validation controls based on a model’s DataAnnotation attributes. I’m a fan of this approach as it makes a lot of sense to WebForm developers. For example, consider the following model:

    public class Booking
    {
        [Required(ErrorMessage = "Client Name is required.")]
        public string ClientName { get; set; }

        [Range(1, 20, ErrorMessage = "Number of Guests must be between 1 and 20.")]
        private public int NumberOfGuests { get; set; }
    }

The generated validators would be very similar to the following:

    <asp:RangeValidator ID="valNumberOfGuests" runat="server" Display="Dynamic"
    ControlToValidate="txtNumberOfGuests" Type="Integer" MinimumValue="1" MaximumValue="20"
    ErrorMessage="Number of Guests must be between 1 and 20."/>

    <asp:RequiredFieldValidator ID="valClientName" runat="server" Display="Dynamic"
    ControlToValidate="txtClientName" ErrorMessage="Client Name is required." />

The other option is to find a way use the ASP.NET MVC validation bits with WebForms. ASP.NET MVC 2 uses the MicrosoftAjax library, while ASP.NET MVC 3 introduced unobtrusive validation with jQuery Validate. This approach would work a lot like the current project, but it would utilize MVC instead of xVal. I have to admit I’m not thrilled about referencing System.Web.Mvc in a WebForms project.

Yet another option is to generate jQuery validation scripts from scratch, using the methods [Dave Ward](http://encosia.com/2009/11/04/using-jquery-validation-with-asp-net-webforms/) has suggested. I like this idea since it doesn’t put a dependency on MVC bits.

It took me a while, but after working with WebForms and trying to make validation better, i.e. more like MVC, I can’t help but think that the best option is to simply move to MVC. But xVal for WebForms can still help projects that can’t be converted to MVC. I’ve already created a [branch](http://xvalwebforms.codeplex.com/SourceControl/list/changesets?branch=nativeWebFormValidation) prototyping the first, most WebForms friendly option. **If you use xVal for WebForms or are interested in attribute based client and server side validation, please let me know which option you would prefer.**


