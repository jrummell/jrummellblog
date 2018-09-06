# MessageBox - The jQuery Plugin


Now that I’ve been working more and more with ASP.NET MVC, I’ve been rewriting some of my server side controls with jQuery plugins. A while back I shared [my version](http://www.jrummell.com/blog/index.php/2008/07/my-version-of-the-messagebox-control/ "My version of the MessageBox control") of Janko’s popular [MessageBox control](http://www.jankoatwarpspeed.com/post/2008/05/28/Create-MessageBox-user-control-using-ASPNET-and-CSS.aspx). I’ve created a similar effect with a jQuery plugin based on the Highlight/Error examples on the jQuery UI [Themes page](http://jqueryui.com/themeroller/).

Here’s an example:

![](http://res.cloudinary.com/jrummell/image/upload/h_84,w_300/v1437489237/message-demo_tywyoe.png "message-demo")

Usage:

    <div id="infoMessage"> 
    	To learn more about ASP.NET MVC visit <a href="http://asp.net/mvc" title="ASP.NET MVC Website"> http://asp.net/mvc</a>. 
    </div>
    <div id="errorMessage" style="display: none"> </div> 
    
    <script type="text/javascript"> 
    	$(document).ready(function () { 
    		$("#infoMessage").message(); 
    		$("#errorMessage").message({ 
    			type: "error", 
    			message: "Oops! An enexpected error has occurred.", dismiss: false }); 
    	}); 
    </script>

The message function accepts the following optional parameters:

- **type**: “info” or “error”. Default is “info”.
- **message**: “your message”. Default is the content of the element.
- **dismiss**: true or false. Default is true. If true, “Click to dismiss” will be appended to the message and clicking the message will hide it.

I’ve created a plugin project at [http://plugins.jquery.com/project/message](http://plugins.jquery.com/project/message) and I’m hosting the source at [http://code.google.com/p/jquery-message/](http://code.google.com/p/jquery-message/).


