# Send a Completed Form Email Without a StringBuilder


 Have you ever had to create a large web form for users to fill out and then receive an email copy after its submitted? That can be tedious work. The first few times I did it, I used a [StringBuilder](http://msdn.microsoft.com/en-us/library/system.text.stringbuilder.aspx) to build the email HTML one control at a time. Later, I viewed the HTML output of the page and replaced all input controls with spans, and then put that HTML in a StringBuilder. Either of these methods work, but it gets real annoying when I later have to add a field or two to the form and therefore to the email HTML.

 I knew there had to be a way to do this programmatically without copying and pasting into a StringBuilder. Well, there is. Here’s a rather common code snippet that does just this:

    public static string GetRenderedHtml(this Control control)
    {
        StringBuilder sbHtml = new StringBuilder();
        using (StringWriter stringWriter = new StringWriter(sbHtml))
        using (HtmlTextWriter textWriter = new HtmlTextWriter(stringWriter))
        {
            control.RenderControl(textWriter);
        }
        return sbHtml.ToString();
    }

 This is great! Let’s try it out on this simple example:

    <div id="divForm" runat="server">
        <fieldset class="inputArea">
        <legend>Contact</legend>
        <asp:Label runat="server" AssociatedControlID="txtName">
        Name</asp:Label>
        <asp:TextBox runat="server" ID="txtName" />
        <asp:Label runat="server" AssociatedControlID="txtEmail">
        Email</asp:Label>
        <asp:TextBox runat="server" ID="txtEmail" />
        <asp:Label runat="server" AssociatedControlID="txtWebsite">
        Website</asp:Label>
        <asp:TextBox runat="server" ID="txtWebsite" />
        <asp:Label runat="server" AssociatedControlID="txtComment">
        Comment</asp:Label>
        <asp:TextBox runat="server" ID="txtComment" TextMode="MultiLine" Rows="4" cols="30" />
        <asp:Button ID="btnSubmit" runat="server" Text="Submit" OnClick="btnSubmit_Click" />
        </fieldset>
    </div>

 
    protected void btnSubmit_Click(object sender, EventArgs e)
    {
        txtRenderedHtml.Text = divForm.GetRenderedHtml();
    }

 Here is what we get:

*Control 'txtName' of type 'TextBox' must be placed inside a form tag with runat=server.*

 So how do you get around that? Well, lets think about this. I’m trying to capture a form and render it as HTML to be included in an email, so I don’t want any [TextBoxes](http://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.textbox.aspx). Lets replace the TextBoxes (and any other editable controls) with [Labels](http://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.label.aspx) and try again.

    public static void ReplaceEditableControls(this Control control)
    {
    // don't bother with controls that aren't visible
    if (!control.Visible)
    {
    return;
    }
    ListControl listControl = control as ListControl;
    IButtonControl buttonControl = control as IButtonControl;
    IValidator validator = control as IValidator;
    IEditableTextControl textControl = control as IEditableTextControl;
    UpdatePanel updatePanel = control as UpdatePanel;
    if (validator != null || buttonControl != null)
    {
    control.Visible = false;
    }
    else if (listControl != null && listControl.SelectedItem != null)
    {
    Label label = new Label {Text = listControl.SelectedItem.Text, CssClass = "text"};
    Replace(listControl, label);
    }
    else if (textControl != null)
    {
    Label label = new Label {Text = textControl.Text, CssClass = "text"};
    Replace((Control) textControl, label);
    }
    else if (updatePanel != null)
    {
    // replace the update panel with a place holder
    PlaceHolder holder = new PlaceHolder();
    Control[] panelControls = new Control[updatePanel.ContentTemplateContainer.Controls.Count];
    updatePanel.ContentTemplateContainer.Controls.CopyTo(panelControls, 0);
    foreach (Control panelControl in panelControls)
    {
    holder.Controls.Add(panelControl);
    }
    ReplaceEditableControls(holder);
    Replace(updatePanel, holder);
    }
    else if (control.HasControls())
    {
    Control[] controlsCopy = new Control[control.Controls.Count];
    control.Controls.CopyTo(controlsCopy, 0);
    foreach (Control controlCopy in controlsCopy)
    {
    ReplaceEditableControls(controlCopy);
    }
    }
    }

 There are a few things to note here.

- The check for [ListControl](http://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.listcontrol.aspx) is before [IEditableTextControl](http://msdn.microsoft.com/en-us/library/system.web.ui.ieditabletextcontrol.aspx) because of the way it implements IEditableTextControl. [ListControl.Text](http://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.listcontrol.text.aspx) returns [ListControl.SelectedValue](http://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.listcontrol.selectedvalue.aspx), but [ListControl.SelectedItem.Text](http://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.listcontrol.selectedvalue.aspx) makes more sense.
- [UpdatePanels](http://msdn.microsoft.com/en-us/library/system.web.ui.updatepanel.aspx) are a special case because of [ContentTemplate](http://msdn.microsoft.com/en-us/library/system.web.ui.updatepanel.contenttemplate.aspx). They are replaced with a [PlaceHolder](http://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.placeholder.aspx) and then the method is recursively called on each child control.
- Finally, if the control has a control collection of its own, a recursive call is made on each child control.
- Notice that the control collection is copied to an array before making the recursive call. This is because the control collection is modified and you can’t modify a collection while iterating it. Well, you can, but you will have problems.

Now we can change the button handler to:

    protected void btnSubmit_Click(object sender, EventArgs e)
    {
        divForm.ReplaceEditableControls();
    }

Which will render the following HTML:

    <div id="divForm">
        <fieldset class="inputArea">
            <legend>Contact</legend>
            <label for="txtName">
            Name</label>
            <span id="txtName" class="text">John Rummell</span>
            <label for="txtEmail">
            Email</label>
            <span id="txtEmail" class="text">jrummell@example.com</span>
            <label for="txtWebsite">
            Website</label>
            <span id="txtWebsite" class="text">john.rummell.info</span>
            <label for="txtComment">
            Comment</label>
            <span id="txtComment" class="text">Check out this new post!</span>
        </fieldset>
    </div>

To capture this as a string, just add a call to GetRenderedHtml:

    protected void btnSubmit_Click(object sender, EventArgs e)
    {
        divForm.ReplaceEditableControls();
        string html = divForm.GetRenderedHtml();
        //TODO: send email
    }

 

 (The form style is a slight variation of [Janko’s tutorial](http://www.jankoatwarpspeed.com/post/2008/07/27/Enhance-your-input-fields-with-simple-CSS-tricks.aspx))