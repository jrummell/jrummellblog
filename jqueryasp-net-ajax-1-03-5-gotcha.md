---
permalink: /jqueryasp-net-ajax-1-03-5-gotcha
title: jQuery/ASP.Net AJAX 1.0/3.5 gotcha
date: 2008-12-18 22:39:00
published: true
tags: javascript
---

Update: If you are curious as to why MS added the .d attribute, find out why at [Encosia](http://encosia.com/2009/02/10/a-breaking-change-between-versions-of-aspnet-ajax/).

I was very frustrated the other day trying to figure out why a jQuery ajax call worked on my dev box but not on the server.It looked something like this:

    json(_serviceUrl, "{}", true,
        function(result) { fillSelect($("#ddlDepartment")[0], result.d); },
        function(ajax) { /* handle error */ });
        
    // calls a json web service
    function json(url, data, async, onSuccess, onFailed)
    {
        $.ajax({
            async: async,
            type: "POST",
            url: url,
            data: data,
            contentType: "application/json; charset=utf-8",
            dataType: "json",
            success: onSuccess,
            error: onFailed
        });
    }

(The code inside the **json** function is straight from [Dave Ward’s](http://encosia.com/) post on [Using jQuery to Consume ASP.NET JSON Web Services](http://encosia.com/2008/03/27/using-jquery-to-consume-aspnet-json-web-services/). Dave’s blog is an excellent resource full of information on jQuery, AJAX, ASP.Net and how to make them play nice together.)

I figured out the problem was **result.d;** on the server it was null, but on my machine it was a ListItem[], as expected. It turns out that on my machine, the web service at _**serviceUrl** was compiled against .Net 3.5. while on the server it was compiled against .Net 2.0 with ASP.NET AJAX Extensions 1.0. In order to get it working on the server, I had to change **result.d** to **result**. Apparently they changed a few things in System.Web.Extensions v3.5. Unfortunately, I’m unable to install .Net 3.5 on the server. So I came up with this helper function to get things working in both 1.0 and 3.5:

    // gets the ajaxResult. Returns ajaxResult.d is if it is not null, else ajaxResult.
    // System.Web.Extensions v3.5 web services will result in ajaxResult.d while v1.0 will be ajaxResult.
    function getResult(ajaxResult)
    {
        return ajaxResult.d == null ? ajaxResult : ajaxResult.d;
    }

So now after replacing **result** with **getResult(result),** the json call looks like this:

    json(_serviceUrl, "{}", true,
        function(result) { fillSelect($("#ddlDepartment")[0], getResult(result)); },
        function(ajax) { /* handle error */ });

Hopefully this will save someone out there some frustration!
