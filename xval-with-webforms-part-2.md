---
permalink: /xval-with-webforms-part-2
title: xVal with WebForms Part 2 
layout: post
date: 2009-08-12 22:17:01
published: true
tags: validation xval-webforms
---


Since my [last post](http://john.rummell.info/john/blog/post/xVal-with-WebForms.aspx), I’ve completely rethought and re-implemented my take on [xVal](http://xval.codeplex.com/) for WebForms. If you’re not familiar with xVal, stop now and read the [tutorial](http://blog.codeville.net/2009/01/10/xval-a-validation-framework-for-aspnet-mvc/). Now that you’re back, lets talk about xVal and WebForms.


## Model

This is the model we’ll be using (you should recognize it from the xVal tutorial):

    public class Booking
    {
        [Required]
        [StringLength(15)]
        public string ClientName { get; set; }

        [Range(1, 20)]
        public int NumberOfGuests { get; set; }

        [Required]
        [DataType(DataType.Date)]
        public DateTime ArrivalDate { get; set; }
    }


## Form

And here is the form:

    <asp:ValidationSummary ID="valSummary" runat="server" />
    <label for="txtClientName">
      Your name:
    </label>
    <asp:TextBox ID="txtClientName" runat="server" />
    <label for="txtNumberOfGuests">
      Number of guests:
    </label>
    <asp:TextBox ID="txtNumberOfGuests" runat="server" />
    <label for="txtArrivalDate">
      Arrival date:
    </label>
    <asp:TextBox ID="txtArrivalDate" runat="server" />
    <asp:Button ID="btnSubmit" runat="server" Text="Submit" OnClick="btnSubmit_Click" />

## ModelValidator

My first try at validation was adding a validator control for each input field. After playing with it a bit, I decided that it would be better to have one validator for the entire model. This control defines the model’s type (ModelType), and then maps each property (PropertyName) to an input control (ControlToValidate).

    <val:ModelValidator ID="valBooking" runat="server" ModelType="xVal.WebForms.Demo.Booking, xVal.WebForms.Demo">
      <ModelProperties>
        <val:ModelProperty PropertyName="ClientName" ControlToValidate="txtClientName" />
        <val:ModelProperty PropertyName="NumberOfGuests" ControlToValidate="txtNumberOfGuests" />
        <val:ModelProperty PropertyName="ArrivalDate" ControlToValidate="txtArrivalDate" />
      </ModelProperties>
    </val:ModelValidator>


## ControlToValidate

The biggest challenge was figuring out how to reference the input controls by given ID instead of by the elementPrefix + PropertyName convention. In other words, MVC xVal assumes that your ClientName input control ID is booking.ClientName, where booking is the elementPrefix and ClientName is the name of the property. This doesn’t work out so well with web forms and generated IDs. I got around this with the ModelProperties collection of ModelValidator. Then I updated the json formatted rule script to include each property’s control ID.


## Complete Source and Demo

Get the [complete source](http://xvalwebforms.codeplex.com/Release/ProjectReleases.aspx?ReleaseId=31487) (with Bookings demo) at the [xVal.WebForms](http://xvalwebforms.codeplex.com/) project page on CodePlex.


