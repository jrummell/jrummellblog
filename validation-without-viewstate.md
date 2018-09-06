---
permalink: /validation-without-viewstate
title: Validation without ViewState
date: 2010-06-01 14:35:21
published: true
tags: validation
---


I ran into an issue today where I needed to validate a few controls without [ViewState](http://msdn.microsoft.com/en-us/library/system.web.ui.control.viewstate.aspx). These were drop down lists whose items were added from the returned value of a [WebService](http://msdn.microsoft.com/en-us/library/t745kdsh.aspx).

    function bindDropDownList(items, controlSelector) {
        var itemsHtml = "<option><\/option>\n";
        $(items).each(function (index, item) {
            itemsHtml += "<option value='" + item.Value + "'>" + item.Text + "<\/option>\n";
        });

        $(controlSelector).html(itemsHtml);
    }

This worked great, until I tried to use a [RequiredFieldValidator](http://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.requiredfieldvalidator.aspx). The client side validation actually worked, but the server side validation kept saying that there was no value selected. This is because the items added by client script aren’t persisted in ViewState, so there is no server side value. This had me stumped for a few minutes. At first I tried replacing the ASP.NET validators with a client side form submit handler, but it was going to be tricky to get the validation messages into the [ValidationSummary](http://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.validationsummary.aspx). I thought about [jQuery Validation](http://docs.jquery.com/Plugins/Validation) for a moment, but I didn’t want to have to deal with two different validation frameworks (ASP.NET and jQuery).

Then it finally dawned on me. It was so simple. All that I needed to do was replace the RequiredFieldValidator with a [CustomValidator](http://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.customvalidator.aspx) and then implement both the client and server side validation methods. I also replaced the [DropDownList](http://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.dropdownlist.aspx) control with a normal HTML select tag since I couldn’t access its selected value from server side code.

    <select id="ddlCounty" name="ddlCounty">
      <option value="">[Select a State]</option>
    </select>
    <asp:CustomValidator ID="valCountyRequired" runat="server"
        ClientValidationFunction="validateCounty"
        OnServerValidate="valCountyRequired_ServerValidate" Display="Dynamic"
        ErrorMessage="The County is required." ValidateEmptyText="true" />
Here’s the client side handler

    function validateCounty(sender, args) {
        args.IsValid = validateRequired(countySelector);
    }

    function validateRequired(selector) {
        var value = $(selector).val();
        return !String.isNullOrEmpty(value);
    }

And the server side handler

    protected void valCountyRequired_ServerValidate(object source, ServerValidateEventArgs args)
    {
        args.IsValid = ValidateRequired(Request.Form["ddlCounty"]);
    }

    private static bool ValidateRequired(string value)
    {
        return !String.IsNullOrEmpty(value);
    }

Notice that I’m not using args.Value in either handler. That’s because I didn’t set [ControlToValidate](http://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.basevalidator.controltovalidate.aspx) on the CustomValidator since without ViewState, it won’t be able to get the value. That’s why I’m using [Request.Form[“ddlCounty”]](http://msdn.microsoft.com/en-us/library/system.web.httprequest.form.aspx) in the server side handler to get the value.

Hope this helps!


