---
permalink: /apply-the-same-css-class-to-all-validators-in-a-web-project
redirect_from: /apply-the-same-css-class-to-all-validators-in-a-web-project/
title: Apply the same CSS class to all validators in a web project 
layout: post
date: 2009-08-16 14:33:01
published: true
tags: css validation
---


I recently had to add a CSS class to all validators in an ASP.NET web application. I started with the theme’s [skin](http://msdn.microsoft.com/en-us/library/ykzx33wh.aspx) file:

<span class="kwrd"><</span><span class="html">asp:CompareValidator</span><span class="attr">runat</span><span class="kwrd">="server"</span><span class="attr">CssClass</span><span class="kwrd">="error"</span><span class="kwrd">/></span><span class="kwrd"><</span><span class="html">asp:CustomValidator</span><span class="attr">runat</span><span class="kwrd">="server"</span><span class="attr">CssClass</span><span class="kwrd">="error"</span><span class="kwrd">/></span><span class="kwrd"><</span><span class="html">asp:RequiredFieldValidator</span><span class="attr">runat</span><span class="kwrd">="server"</span><span class="attr">CssClass</span><span class="kwrd">="error"</span><span class="kwrd">/></span><span class="kwrd"><</span><span class="html">belCommon:ZipCodeValidator</span><span class="attr">runat</span><span class="kwrd">="server"</span><span class="attr">CssClass</span><span class="kwrd">="error"</span><span class="kwrd">/></span><span class="kwrd"><</span><span class="html">belCommon:PhoneNumberValidator</span><span class="attr">runat</span><span class="kwrd">="server"</span><span class="attr">CssClass</span><span class="kwrd">="error"</span><span class="kwrd">/></span>

But what if I decide to use another validator down the road? I would have to remember to add it to the skin. Knowing that I was bound to forget, I sought out another method. After doing some digging, I found that ASP.NET generates a JavaScript variable called Page_Validators. This is an array of all the validator span elements on the current page. Now that I have access to the spans, I could write a script in the site’s [Master Page](http://msdn.microsoft.com/en-us/library/wtxbf3hh.aspx) to apply the class:

<span class="kwrd">if</span> (Page_Validators != <span class="kwrd">null</span>) { <span class="kwrd">for</span> (i = 0; i < Page_Validators.length; i++) { Page_Validators[i].className = <span class="str">"error"</span>; } }

To have it run when the page is loaded, I added it as an [Sys.Application.init](http://msdn.microsoft.com/en-us/library/bb397532.aspx) handler:

Sys.Application.add_init(<span class="kwrd">function</span>(sender, args) { <span class="kwrd">if</span> (Page_Validators != <span class="kwrd">null</span>) { <span class="kwrd">for</span> (i = 0; i < Page_Validators.length; i++) { Page_Validators[i].className = <span class="str">"error"</span>; } } });

You could also use jQuery’s [document.ready](http://docs.jquery.com/Events/ready#fn) handler:

$(document).ready(<span class="kwrd">function</span>() { <span class="kwrd">if</span> (Page_Validators != <span class="kwrd">null</span>) { <span class="kwrd">for</span> (i = 0; i < Page_Validators.length; i++) { Page_Validators[i].className = <span class="str">"error"</span>; } } });


