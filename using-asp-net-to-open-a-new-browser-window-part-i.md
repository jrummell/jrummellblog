# Using ASP.Net to open a new browser window – Part I


I recently needed to add a ‘pop-up’ window to a website. Now I’m not a fan of javascript, never have been. I like my code to be type safe and compile. So I came up with a C# helper class to do the dirty work for me. I must give credit to where I found the idea. I saw something like this in Chapter 4 of <span style="font-style: italic;">Developing Web Applications with Microsoft VB .Net and VC#</span> .Net by Jeff Webb and MS Corp. In this book they create BrowserWindow class and use it inside some javascript. I’ve take a different approach, so that I can avoid having to write the javascript in the future.

I created a static helper class that contains the following method. I also provided some methods that overload this, but you get the idea.

{% raw %}
``` csharp
    public static void RegisterOpenWindowScript(Page page, string key, string url, WindowName name, int width,
        int height, int left, int top, bool location, bool menubar, bool resizable, bool scrollbars, bool status,
        bool titlebar)
    {
        if (!page.ClientScript.IsClientScriptBlockRegistered(typeof (OpenWindowHelper), key))
        {
            string script =
                String.Format(
                    @" function {12}() {{ var win = window.open('{0}', '{1}', 'width={2},height={3},left={4},top={5},location={6}, menubar={7},resizable={8},scrollbars={9}, status={10},titlebar={11}'); win.focus(); }} ",
                    url, name, width, height, left, top, GetInt(location), GetInt(menubar), GetInt(resizable),
                    GetInt(scrollbars), GetInt(status), GetInt(titlebar), openFunctionName);
            page.ClientScript.RegisterClientScriptBlock(typeof (OpenWindowHelper), key, script, true);
        }
    }
```
{% endraw %}

WindowName is an enum that defines the possible window names: _blank, _parent, _self, _top. GetInt() simply returns 1 if true and 0 if false.

As I’m writing this I’m realizing that it would be a cool idea to merge this static class with the BrowserWindow class. You could define a BrowserWindow object and perform the Open() operation to open the window. Hmm … I’m seeing a follow up post in the near future.
