---
layout: posts
title: Automating web browsing using PowerShell
date: 2015-01-09 17:58
author: edward
categories: blog PowerShell, Scripting
tags: automation, browser, Selenium
slug: automating-web-browsing-using-powershell
status: published
---

A lot of our Production Quality Control (QC) operations where I work require checking that data has been uploaded to one of our websites, using either one of our internal tools, or our backdoor access to one of our customer-facing sites. This is all right when we're checking a couple of customer jobs, but gets tedious VERY quickly for routine QC of dozens or hundreds of customer jobs.

A web app works well as a manual tool ("enter text to be searched for in this box, click Search, click the link for desired item in the list of items matching your search term..."), but our internal tools and customer-facing sites were never designed to be scripted.

For a while, when I was working with more cross-platform scripting languages, I was looking at [Selenium](http://www.seleniumhq.org/). Selenium allows you to [control popular web browsers from a number of programming languages](http://www.seleniumhq.org/about/platforms.jsp#programming-languages), including Python, Ruby, and C# - but not directly from PowerShell. It would be possible to write my own PowerShell wrapper for C# to control Selenium, but I don't have any experience extending PowerShell with C#, and since we're not a C# shop, I think that would be very fragile from a long-term maintenance standpoint.

Anyway, unlike the typical Selenium application, our Production QC ops aren't testing a single web app across multiple browsers. We're searching for a multitude of data items, but we only have to find each one once, in a single web browser. A more robust solution would be to use something more native to PowerShell to control a single browser - which could even be Internet Explorer (perish the thought!).

I Googled "powershell web browser automation" and came up with a number of possibilities.

[Web UI Automation with Windows PowerShell](http://msdn.microsoft.com/en-us/magazine/cc337896.aspx "MSDN article - Web UI Automation with Windows PowerShell"), is an MSDN article from 2008 and talks about using COM to control Internet Explorer, which is something I've dabbled in using VB Script. My first experiment with the method wasn't successful, though, so I looked for troubleshooting info for COM in my handy copy of Windows PowerShell in Action. As it happens, the book illustrates COM with an example of "...a COM-based tool to help manage...browser windows," so the book probably offers a more fertile field for further research.

[A post on StackOverflow](http://stackoverflow.com/questions/594831/ie-automation-with-windows-powershell) then led me to [WatiN](http://watin.org/) - Web Application Testing in .Net. WatiN allows control of Internet Explorer AND Firefox, so it might be even better than using COM.
