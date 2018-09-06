---
permalink: /confluence-universal-wiki-converter-uwc-for-screwturn
title: Confluence Universal Wiki Converter (UWC) for ScrewTurn
date: 2010-09-02 10:44:46
published: true
tags: confluence-uwc wiki
---


My [company](http://www.beldenbrick.com) recently started a corporate wiki. We initially used [ScrewTurn](http://www.screwturn.eu/), an excellent open source ASP.NET application. It worked great for our department, but it wasn’t taking off for the rest of the company. Non-technical users just couldn’t get wiki markup and weren’t able to manage edits with the visual editor. So we turned to [Confluence](http://www.atlassian.com/software/confluence/), an enterprise class commercial wiki. I won’t get into a feature comparison here. There are plenty of [other](http://www.wikimatrix.org/) [sites](http://en.wikipedia.org/wiki/Comparison_of_wiki_software) devoted to that.

Since we had been using ScrewTurn for several months, we had a few hundred pages that we needed to migrate to Confluence. We hoped to find a utility to take care of the migration, but we were unable to find one. So in the end, I wrote my own UWC implementation for ScrewTurn.

The [Confluence Universal Wiki Converter (UWC)](https://studio.plugins.atlassian.com/wiki/x/H4Mi) is defined as

> … a standalone application which converts pages from other wikis to Confluence. It’s also an extensible framework, allowing users with wikis that aren’t currently supported to add their own converters.

Here are the general steps I took to implement a wiki converter for ScrewTurn.

1. [Implement a Wiki Converter for ScrewTurn](https://studio.plugins.atlassian.com/wiki/display/UWC/UWC+Developer+Documentation#UWCDeveloperDocumentation-ImprovinganExistingWikiConverter). I used MediaWiki’s Syntax Converter as a base since the basic wiki syntax is very similar. I also implemented a few Converter classes, [UserDateConverter](https://studio.plugins.atlassian.com/wiki/display/UWC/UWC+UDMF+Framework) (requires the [Confluence UDMF plugin](https://plugins.atlassian.com/plugin/details/17666)), [PagenameConverter](https://studio.plugins.atlassian.com/wiki/display/UWC/UWC+Page+Titles+Framework), [AttachmentsConverter](https://studio.plugins.atlassian.com/wiki/display/UWC/UWC+Attachments+Framework), MetaDataCleaner (to remove the first three lines in ScrewTurn page files that include page name, date, and ##PAGE##).
2. In ScrewTurn, [change the page storage provider to Local Pages Provider](http://www.screwturn.eu/Help.DataMigration.ashx) (if its using a different provider such as SQL).
3. Run the customized UWC implemented in step 1 and convert one namespace at a time.

You can also view my [StackOverflow question and answer](http://stackoverflow.com/q/2830227/26226) that inspired this post.

[Confluence UWC with ScrewTurn converter source code](https://bitbucket.org/jrummell/confluenceuwc-screwturn)