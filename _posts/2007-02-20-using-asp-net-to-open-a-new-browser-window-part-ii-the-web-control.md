---
permalink: /using-asp-net-to-open-a-new-browser-window-part-ii-the-web-cont
redirect_from: /using-asp-net-to-open-a-new-browser-window-part-ii-the-web-cont/
title: Using ASP.Net to open a new browser window – Part II, the web control
date: 2007-02-20 13:24:00
published: true
tags: javascript
---



This a follow up to my previous post, [Using ASP.Net to open a new browser window – Part I](http://www.jrummell.com/blog/index.php/2007/02/using-asp-net-to-open-a-new-browser-window-part-i/ "Using ASP.Net to open a new browser window – Part I").  
 There I used a static helper class to register a javascript function to open a new browser window. I’ve scratched the static class and replaced it with a BrowserWindow class and a PopUpWindow WebControl. BrowserWindow simply encapsulates all of the parameters passed to RegisterOpenWindowScript(), and PopUpWindow registers the javascript function and the call to the function.

Here’s an example:

<span class="kwrd"><</span><span class="html">cc1:PopUpWindow</span><span class="attr">ID</span><span class="kwrd">="PopUpWindow1"</span><span class="attr">runat</span><span class="kwrd">="server"</span><span class="attr">OpenOnLoad</span><span class="kwrd">="false"</span><span class="kwrd">/></span><span class="kwrd"><</span><span class="html">asp:Button</span><span class="attr">ID</span><span class="kwrd">="Button1"</span><span class="attr">runat</span><span class="kwrd">="server" </span><span class="attr">Text</span><span class="kwrd">="Open Window"</span><span class="attr">OnClick</span><span class="kwrd">="Button1_Click"</span><span class="kwrd">/></span>

<span class="kwrd">protected</span><span class="kwrd">void</span> Page_Load(<span class="kwrd">object</span> sender, EventArgs e) { BrowserWindow window = <span class="kwrd">new</span> BrowserWindow( <span class="str">"http://john.rummell.info/blog"</span>, 800, 600); window.Resizable = <span class="kwrd">true</span>; window.Scrollbars = <span class="kwrd">true</span>; PopUpWindow1.BrowserWindow = window; } <span class="kwrd">protected</span><span class="kwrd">void</span> Button1_Click(<span class="kwrd">object</span> sender, EventArgs e) { PopUpWindow1.OpenWindow(); }

PopUpWindow exposes a few of BrowserWindow’s properties: Width, Height, and Url. For more options, create a BrowserWindow object and set PopUpWindow’s  BrowserWindow property with it, as shown in Page_Load. You can use the OpenWindow() method of PopUpWindow to open the window as shown in Button1_Click, or you can set OpenOnLoad to true to have it open when the page loads.
