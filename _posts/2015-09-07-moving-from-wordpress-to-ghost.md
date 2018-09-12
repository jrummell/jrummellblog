---
permalink: /moving-from-wordpress-to-ghost
redirect_from: /moving-from-wordpress-to-ghost/
title: Moving from WordPress to Ghost 
layout: post
date: 2015-09-07 15:59:01
published: true
tags: blogging
---

I recently moved my blog from WordPress to Ghost. The Ghost installation was breeze thanks to [Ghost Azure](https://github.com/AzureWebApps/Ghost-Azure), as shared by [Scott Hanselman](http://www.hanselman.com/blog/UPDATEDFor2015HowToInstallTheNodejsGhostBlogSoftwareOnAzureWebAppsAndTheDeployToAzureButton.aspx).

[Migrating content](https://www.ghostforbeginners.com/how-to-transfer-blog-posts-from-wordpress-to-ghost/) was also fairly easy.

Now that I had a new azure site with all of my content, the fun part began. The ghost exporter does a decent job at converting html to markdown, but it misses a few things. I had a correct spacing in a few places, but not many. Now, the source code embedded in my posts was a complete mess. I highly recommend keeping your wordpress blog up until you finish the migration process. What worked best for me was to browse to the old wordpress post with the source code, copy/paste the source code into a text editor, and format as needed. Using Visual Studio to format the code, then highlight all and hit tab was enough in most cases. Then I copy/pasted that into the ghost post and everything was great.

For syntax highlighting I used [highlight.js as described here](http://massimilianomarini.com/how-to-use-highlightjs-into-ghost-blogging-platform/).

Now that the content was ready to go, I needed to redirect the wordpress fromat urls to the ghost format. In wordpress I was using `/blog/year/month/post-title`, while ghost uses `/post-title/`. URL Rewrite to the rescue! Well, mostly.

``` xml
    <rule name="wordpress to ghost - posts" stopProcessing="true">
        <match url="^blog/(index\.php/)?\d+/\d+/([\w\-]+)/?" />
        <action type="Redirect" url="{R:2}" />
    </rule>
    <rule name="wordpress to ghost - pages" stopProcessing="true">
        <match url="^blog/([\w\-]+)/?" />
        <action type="Redirect" url="{R:1}" />
    </rule>
    <rule name="/blog to /" stopProcessing="true">
        <match url="^blog/?" />
        <action type="Redirect" url="/" />
    </rule>
```

The first rule redirects all posts, the second pages, and the third redirects `/blog` to `/`, since my old site ran in a virtual directory and ghost does not.

The first rule works just fine for urls such as `http://www.jrummell.com/blog/2011/03/the-state-of-xval-for-webforms/`. However, urls with index.php redirect to `/index/` for some reason that I can't explain. I have some older stackoverflow posts with links using index.php in the url that are now broken, such as `http://www.jrummell.com/blog/index.php/2011/03/the-state-of-xval-for-webforms/`. I've turned to [stackoverflow](http://stackoverflow.com/questions/32442293/redirecting-wordpress-urls-in-a-ghost-site-hosted-in-azure) for a solution, and I'll update this if I find one.

**Update**

Thanks to Ryan Joy on stackoverflow, I got the posts rule working. Here's the correct regex that escapes forward slashes:

``` xml
    <rule name="wordpress to ghost - posts" stopProcessing="true">
        <match url="^blog\/(index\.php\/)?\d+\/\d+\/([\w\-]+)\/?" />
        <action type="Redirect" url="{R:2}" />
    </rule>
```