---
permalink: /a-sitemapprovider-for-static-web-sites
title: A SiteMapProvider for Static Web Sites 
layout: post
date: 2007-08-03 20:34:00
published: true
tags: 
---


The new navigation features of ASP.Net 2.0 are pretty cool. If you haven’t seen them yet, check out [ScottGu’s blog](http://weblogs.asp.net/scottgu/archive/2006/02/14/438241.aspx) for more information.

I’ve  
 seen a few blog posts on SiteMapProvider implementations for dynamic  
 web sites, but not a whole lot on providers for static web sites. Sure,  
 you could use the default implementation and manually update the  
 web.sitemap xml file, but what about large sites? In my opinion, its  
 not worth the effort.

Here are my requirements for a static SiteMapProvider:

- Must automagically update whenever pages are added/removed.
- Must be able to only include specified file types.
- Must be able to exclude specified directories under the application’s virtual directory.

So I went searching and didn’t find anything. The closest I found was a [macro by K. Scott Allen](http://odetocode.com/Blogs/scott/archive/2005/11/29/2537.aspx)  
 that generates a web.sitemap file from a web project. A noble effort,  
 but I needed a bit more. So I set off to implement my own provider.  
 Using the [SqlSiteMapProvider example](http://msdn2.microsoft.com/en-us/library/Aa479033.aspx) as a reference, I had created my own **StaticFileSiteMapProvider** by lunch time.

The  
 implementation is rather straighforword. It starts at the application  
 path (~/) and recurses each of its sub directories. There is a **FileExtensions** property that defines the types of files to include (e.g. aspx, html) and there is also a **DirExclusions** property that defines the directory name patterns to exclude (e.g. bin, App_*). The **DefaultDocuments** property defines the default document names for a directory (e.g index, default).

Why do I need a **DefaultDocuments**  
 property? I can answer that with another question. What happens when  
 you’ve got a directory in your app that doesn’t have an index page?  
 Well, the provider will generate a link to that folder, but clicking on  
 it will result in a Directory Listing Denied error (at least I hope you  
 would have your site set up that way). In  
 BuildSiteMap(SiteMapNode  
 parentNode, string directory), if the current node’s directory doesn’t  
 have a default document page, then the node’s url isn’t set, ensuring  
 that its not hyperlinked.

On to the code. I’ve included the main parts of the class below. For a full listing, use the link at the end of this post.

    public override SiteMapNode BuildSiteMap()
    {
        lock (this)
        {
            if (isBuilt)
            {
                return root;
            }
            string physicalAppPath = HttpContext.Current.Server.MapPath("~/");
            BuildSiteMap(null, physicalAppPath);
            isBuilt = true;
            return root;
        }
    }

    /// <summary>
    /// Recursive method to build the site map.
    /// </summary>
    /// <param name="parentNode">The parent node.</param>
    /// <param name="directory">The directory.</param>
    private void BuildSiteMap(SiteMapNode parentNode, string directory)
    {
        // create the current node
        string url = GetUrlFromPhysicalPath(directory);
        string title = parentNode == null ? "Home" : Path.GetFileName(directory);
        SiteMapNode node = new SiteMapNode(this, url, url, title);
        // set the root
        if (parentNode == null)
        {
            root = node;
        }
        // add a node foreach file
        string[] files = GetFiles(directory);
        foreach (string file in files)
        {
            url = GetUrlFromPhysicalPath(file);
            SiteMapNode fileNode = new SiteMapNode(this, url, url, Path.GetFileNameWithoutExtension(file));
            AddNode(fileNode, node);
        }
        // unset the url if there isn't an index file in the directory
        if (!Array.Exists(files, delegate(string match)
        {
            foreach (string index in DefaultDocuments.Split(','))
            {
                if (String.Compare(Path.GetFileNameWithoutExtension(match), index.Trim(),
                    StringComparison.OrdinalIgnoreCase) == 0)
                {
                    return true;
                }
            }
            return false;
        }))
        {
            // Note: setting node.Url to null doesn't change the value, so I'm setting it to String.Empty, 
            // which is its default value
            node.Url = String.Empty;
        }
        // recurse sub directories
        string[] directories = GetDirectories(directory);
        foreach (string dir in directories)
        {
            BuildSiteMap(node, dir);
        }
        // only add the current node if it has children
        // Note: node.HasChildren throws an InvalidOperationException, so I'm checking the 
        // file and directory arrays instead
        if (files.Length > 0 && directories.Length > 0)
        {
            AddNode(node, parentNode);
        }
    }

web.config settings:

    <siteMap defaultProvider="StaticFileSiteMapProvider">
      <providers>
        <add name="StaticFileSiteMapProvider" type="Providers.StaticFileSiteMapProvider"
        fileExtensions="asp, htm"
        defaultDocuments="index"
        dirExclusions="bin, obj, Properties, App_*, DMS, old" />
      </providers>
    </siteMap>