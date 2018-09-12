---
permalink: /sql-server-management-studio-and-tortoisesvn
redirect_from: /sql-server-management-studio-and-tortoisesvn/
title: SQL Server Management Studio and TortoiseSVN 
layout: post
date: 2009-11-18 23:23:17
published: true
tags: SQL svn
---


**Update**: SQL Source Control was released a while back! See my article on [Simple-Talk](http://www.simple-talk.com/sql/sql-tools/sql-source-control---no-more-database-development-without-it/) for more information.

At work we maintain a few [SQL Server Management Studio](http://msdn.microsoft.com/en-us/library/ms174173.aspx) (SSMS) solutions for our SQL views, stored procedures and functions. We also use [TortoiseSVN](http://tortoisesvn.tigris.org/) for source control. Unfortunately, there are no SVN add-ins for SSMS and the ones for Visual Studio don’t work ([VisualSVN](http://www.visualsvn.com/visualsvn/), [AnkhSVN](http://ankhsvn.open.collab.net/)). Its a bit frustrating that SSMS is built on the same technology as Visual Studio, but lacks so many of the features that I’ve grown accustomed to, such as the Add-in Manager.

[Red Gate](http://www.red-gate.com/), however, is currently working on a add-in called [SQL Source Control](http://www.red-gate.com/products/SQL_Source_Control/) with a planned release in 2010. But what to do until then? Well, there is one officially supported point of extensibility in SSMS: [External Tools](http://msdn.microsoft.com/en-us/library/ms177402.aspx). Here are a few that I’ve been using with TortoiseSVN lately:

    Title: SVN Commit  
    Command: C:\Program Files\TortoiseSVN\bin\TortoiseProc.exe  
    Arguments: /Command:commit /path:"$(SolutionDir)"
    Initial directory: $(SolutionDir)

    Title: SVN Update  
    Command: C:\Program Files\TortoiseSVN\bin\TortoiseProc.exe  
    Arguments: /Command:update /path:"$(SolutionDir)"  
    Initial directory: $(SolutionDir)

    Title: SVN Log (Solution)  
    Command: C:\Program Files\TortoiseSVN\bin\TortoiseProc.exe  
    Arguments: /Command:log /path:"$(SolutionDir)"
    Initial directory: $(SolutionDir)

    Title: SVN Log (Current Item)  
    Command: C:\Program Files\TortoiseSVN\bin\TortoiseProc.exe  
    Arguments: /Command:log /path:"$(ItemFileName)$(ItemExt)"  
    Initial directory: $(ItemDir)

    Title: SVN Diff  
    Command: C:\Program Files\TortoiseSVN\bin\TortoiseProc.exe  
    Arguments: /Command:diff /path:"$(ItemFileName)$(ItemExt)"  
    Initial directory: $(ItemDir)
