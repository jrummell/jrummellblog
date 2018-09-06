---
permalink: /my-version-of-the-messagebox-control
title: My version of the MessageBox control
date: 2008-07-09 14:46:00
published: true
tags: 
---


**Update**: I fixed a couple issues and updated the zip file below.

- The close button’s client side onclick event wasn’t getting the correct client id when used inside a Master page.I moved the onclick addition to the Load event to correct it.
- The message would not display when used inside an UpdatePanel.The Page.ClientScript.RegisterStartupScript method only works for synchronous (non-AJAX) post backs.The control now uses the ScriptManager.RegisterStartupScript method for async post backs.

Last month [Janko](http://www.jankoatwarpspeed.com) posted an [excellent article](http://www.jankoatwarpspeed.com/post/2008/05/28/Create-MessageBox-user-control-using-ASPNET-and-CSS.aspx) about creating standard website message boxes.He concluded it with this:

> I am aware that this code isn’t perfect, although it did a great job for me on several projects. Did you ever need or do something similar? Do you have any other ideas how this could be implemented? Share it!

I took that as a challenge. I loved his idea but the implementation didn’t fit my needs 100%. Since I maintain multiple web projects I try to use [custom web controls](http://msdn.microsoft.com/en-us/library/yhzc935f.aspx) over [user controls](http://msdn.microsoft.com/en-us/library/3457w616.aspx). I find it easier to reference a web control library than keeping track of .ascx files.In addition, I found that if a message was displayed and the user clicks the close button and then does something else on the page that initiates a postback, the message would be visible again.

### Creating the Custom WebControl

I used [CompositeControl](http://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.compositecontrol.aspx) as my base class and started by duplicating the functionality of Janko’s user control.The first difference in the code is the [CreateChildControls](http://msdn.microsoft.com/en-us/library/system.web.ui.control.createchildcontrols.aspx) method.

    protected override void CreateChildControls()
    {
        base.CreateChildControls();

        // the wrapper div
        Style[HtmlTextWriterStyle.Display] = "none";
        CssClass = "container";

        // the message div
        _pnlMessageBox = new Panel();
        _pnlMessageBox.ID = "pnlMessageBox";
        Controls.Add(_pnlMessageBox);

        // the close button
        _hlClose = new HyperLink();
        _hlClose.ID = "hlClose";
        _hlClose.ToolTip = "Close";
        _hlClose.CssClass = "close";
        // add the onclick event
        string onclick = String.Format("document.getElementById('{0}').style.display = 'none';", ClientID);
        _hlClose.Attributes.Add("onclick", onclick);
        _pnlMessageBox.Controls.Add(_hlClose);

        // the message
        _pMessage = new HtmlGenericControl("p");
        _pnlMessageBox.Controls.Add(_pMessage);

        _litMessage = new Literal();
        _litMessage.ID = "litMessage";
        _pMessage.Controls.Add(_litMessage);
    }

This is where I’m (you guessed it) creating and adding the child controls.One difference here is *Style[HtmlTextWriterStyle.Display] = “none”;*. I’ll get to that next.Another difference is I’m not using an image control inside the close hyper link. I modified the css to include a background image for the hyperlink instead.I did this because I didn’t want to worry about setting the close image url in each project in addtion to including the css file and associated images.

### Keeping the MessageBox Hidden on PostBack

Instead of using the server side Visible property to show/hide the message I’m using javascript and css.This can be better or worse.Better because it allows me to keep the message hidden on post back, and worse because the html bits are sent to the client on every request even if the message is hidden.In the CreateChildControls method above I’m setting the style display value to ‘none’.This is what hides message.In the Show method, I’m using a startup script to change the display value to its default – ‘block’, which shows the message.This startup script is lost on post back so the message goes back to hiding.

    // add a script to display the message on page load
    string script = String.Format("document.getElementById('{0}').style.display = 'block';", ClientID);

    Page.ClientScript.RegisterStartupScript(GetType(), ClientID, script, true);

I’ve been using this is production for a few weeks now and its done very well for me, but I also don’t claim to write perfect code. If you have found a better way, please let me know!


