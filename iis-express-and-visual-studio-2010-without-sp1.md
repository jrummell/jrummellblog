---
permalink: /iis-express-and-visual-studio-2010-without-sp1
title: IIS Express and Visual Studio 2010 without SP1 
layout: post
date: 2011-03-01 16:20:07
published: true
tags: iis
---


Here’s how you can integrate [IIS Express](http://learn.iis.net/page.aspx/868/iis-express-overview/) with Visual Studio 2010 without SP1. I’m taking advantage of External Tools again. There are two very simple ways to run IIS Express from the [command line](http://learn.iis.net/page.aspx/870/running-iis-express-from-the-command-line/). The first is to pass the web project path:

[![](http://res.cloudinary.com/jrummell/image/upload/h_296,w_300/v1437490601/path_xcq4xl.png "IIS Express with Path")](http://res.cloudinary.com/jrummell/image/upload/v1437490601/path_xcq4xl.png)

You can use this by selecting your web project in the solution explorer and then running the tool.

The second way is to use a configuration file. This is for when you needsomethingdifferent than the default settings. For example, if you need to enable PHP. Save your applicationHost.config in the root of your project file before running this:

[![](http://res.cloudinary.com/jrummell/image/upload/h_296,w_300/v1437489241/config_hne7fs.png "IIS Express with Config")](http://res.cloudinary.com/jrummell/image/upload/v1437489241/config_hne7fs.png)

Now that we can run IIS Express, we need to configure the project to use it. In the Web tab of your project settings, select Use Custom Web Server and type the Server Url that IIS Express is configured to use. The default is http://localhost:8080/.

[![](http://res.cloudinary.com/jrummell/image/upload/h_176,w_300/v1437490620/websettings_ong3mc.png "Project Web Settings")](http://res.cloudinary.com/jrummell/image/upload/v1437490620/websettings_ong3mc.png)

Wasn’t that easy?