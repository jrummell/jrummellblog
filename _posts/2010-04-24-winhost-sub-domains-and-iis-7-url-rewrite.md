---
permalink: /winhost-sub-domains-and-iis-7-url-rewrite
title: WinHost, Sub-Domains, and IIS 7 URL Rewrite
date: 2010-04-24 18:31:40
published: true
tags: iis
---

I switched my web host from GoDaddy to WinHost this week. I only I ran into two issues. The first was how WinHost handles sub-domains. GoDaddy’s sub-domains point to a sub folder of the same name, e.g. http://sub.example.com points to \sub. You can then access sub-domain content from http://sub.example.com/abc or http://sub.example.com/sub/abc. There is no way that I could find to enforce either /abc or /sub/abc and often after arriving to the site via /abc you would be linked to /sub/abc.

WinHost doesn’t automatically redirect your sub-domains; they leave that up to the customer. They suggest using custom code on the default page to redirect based on the server name Request variable. This would be fine if users always went to that page first, but what if you already have links to http://sub.example.com/somepage.aspx? They will be broken, of course!

WinHost allows you to manage IIS7 through the Remote IIS Manager and includes the URL Rewrite Module, so I thought I’d give that a try. My first attempt was to rewrite http://john.rummell.info to /john using the information provided by Ben Powell on his blog:

``` xml
    <rule name="john.rummell.info" stopProcessing="true">
        <match url="(.*)" />
            <conditions>
                <add input="{HTTP_HOST}" pattern="^john.rummell.info$" />
            </conditions>
        <action type="Rewrite" url="john/{R:1}" />
    </rule>
```

This works if you use only absolute paths, but it broke all of my relative css links and form actions. This is because even though you never see /john in the url, Request.ApplicationPath is still /john/somepage.aspx. I decided that this wasn’t right path to take. Then I tried a similar rule that would redirect instead of rewrite:

``` xml
    <rule name="john.rummell.info" stopProcessing="true">
        <match url="(.*)" />
        <conditions logicalGrouping="MatchAll">
            <add input="{HTTP_HOST}" pattern="^john.rummell.info$" />
            <add input="{REQUEST_URI}" negate="true" pattern="/john" />
        </conditions>
        <action type="Redirect" url="http://john.rummell.info/john/{R:1}" />
    </rule>
```

This will redirect all requests from http://john.rummell.info to http://john.rummell.info/john. The first input is the same as the rewrite rule. The second input makes sure that the path doesn’t already start with /john. This prevents it from redirecting infinitely (/john/john/john/john etc). Its not ideal since the sub domain appears in the url twice, but at least its guaranteed to always be that way, unlike GoDaddy.

The other issue I had was that using the Facebook Developer Toolkit and reCaptcha both threw a SecurityException. This was an easy fix once I realized that I could change the trust level in IIS. Changing it to High fixed the problem. So far I’ve been very pleased!

So far I’ve been very pleased with WinHost. They have reasonable pricing, their support team has been quick to respond, and they allow you to manage your site with IIS Manager and SQL Server Management Studio. Good bye GoDaddy!