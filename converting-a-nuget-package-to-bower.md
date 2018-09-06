---
permalink: /converting-a-nuget-package-to-bower
title: Converting a nuget package to Bower
date: 2015-12-12 01:06:35
published: true
tags: nuget bower
---

If you're an ASP.NET developer like me, you've come to know and love NuGet as your go to resource for managing external libraries. It has everything from ASP.NET MVC and Entity Framework, to jQuery and bootstrap. You may have heard that Visual Studio 2015 has [out of the box support](https://msdn.microsoft.com/en-us/magazine/mt573714.aspx) for [Bower](http://bower.io/), a client side package manager for the web. It seems that Bower (and NPM) is what everyone else outside of the Microsoft ecosystem has been using for a while now, and Visual Studio is finally catching up.

I have a handful of open source projects on nuget.org, and some of them are client side web libraries. Recently I converted one to bower. This post will demonstrate the differences I've found between the two package managers and show you how to create a bower package from an existing nuget package.

## NuGet Package Authoring

I'm assuming if you're reading this you're already familiar with [this process](http://docs.nuget.org/Create/Creating-and-Publishing-a-Package), but I'll give you a brief summary. One you have an ASP.NET web project ready to go:

- Open the command line in your project root (next to your .csproj file)
- Run `nuget spec` to create the .nuspec file
- Open the .nuspec file, include any files you want in your package and specify your project's nuget dependencies
- Run `nuget pack MyProject.nuspec -Version 1.2.3` to create the package
- Run `nuget push MyProject.1.2.3.nupkg` to publish it to nuget.org

Then, when you need to update your package, you repeat the pack and push steps with a new version number.

## Bower Package Authoring

Bower takes a different approach. Really, the only thing that's similar to nuget is [bower.json](https://github.com/bower/spec/blob/master/json.md), the .nuspec equivalent.

The first major difference I noticed was that bower.json must be in the git repository root. Not the project root, the repository root. The bower process is as follows:

- Run `npm install bower` to install bower (only required if you want to manage bower from the command line)
- Install any bower packages that are dependencies for your project. There are a few ways to do this:
 - Right click the project in Visual Studio and select "Manage Bower Packages",
 - Edit the bower.json file directly, or
 - Run `bower install <package>` from the command line

If you haven't already, push your project to a public git repository, like GitHub, making sure to push your tags. Tags should follow the [semver](http://semver.org/) guidelines.

The next step is to register your project with bower:

    bower register my-project git://github.com/user/my-project.git

Congratulations, your project is now on bower. When you need to publish a new version, there's no need to re-register or push any updates to bower. All you need to do is push a new version tag to your remote repository.

## Comparing the two

Here's a simple, real world comparison of nuget vs bower for [bootstrap.message](https://github.com/jrummell/bootstrap.message).

### .nuspec vs bower.json

Sample .nuspec (NuGet):

    <?xml version="1.0"?>
    <package xmlns="http://schemas.microsoft.com/packaging/2010/07/nuspec.xsd">
        <metadata>
            <id>Bootstrap.Message</id>
            <version>$version$</version>
            <authors>John Rummell</authors>
            <owners>John Rummell</owners>
            <licenseUrl>http://www.gnu.org/licenses/gpl.html</licenseUrl>
            <projectUrl>https://github.com/jrummell/bootstrap.message</projectUrl>
            <requireLicenseAcceptance>false</requireLicenseAcceptance>
            <description>
            A simple jQuery plugin that uses the Bootstrap alert css classes to display info and error messages.
            </description>
            <tags>jquery bootstrap plugin message</tags>
            <dependencies>
                <dependency id="bootstrap" version="3.3.6" />
            </dependencies>
        </metadata>
        <files>
            <file src="dist\js\*.js" target="content\Scripts"/>
        </files>
    </package>


Sample bower.json (Bower):

    {
        "name": "bootstrap.message",
        "description": "A simple jQuery plugin that uses the Bootstrap alert css classes to display info and error messages.",
        "authors": [
            {
                "name": "John Rummell",
                "homepage": "http://jrummell.com"
            }
        ],
        "main": "bootstrap.message.js",
        "moduleType": [],
        "license": "http://www.gnu.org/licenses/gpl.html",
        "repository": {
            "type": "git",
            "url": "https://github.com/jrummell/bootstrap.message.git"
        },
        "keywords": [
            "bootstrap",
            "message",
            "jquery",
            "plugin"
        ],
        "dependencies": {
            "bootstrap": "^3.3.6"
        },
        "ignore": [
        "**/.*",
        "node_modules",
        "bower_components"
    ]
    }

### Publishing

nuget command line (NuGet):

    nuget pack bootstrap.message.nuspec -Version 1.10.2
    nuget push -Version bootstrap.message.1.10.2.nupkg

git command line (Bower):

    git tag -a v1.10.2 -m "Version 1.10.2"
    git push --tags