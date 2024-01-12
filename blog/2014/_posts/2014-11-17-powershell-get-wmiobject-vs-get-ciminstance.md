---
layout: posts
title: PowerShell - Get-WmiObject vs. Get-CimInstance
date: 2014-11-17 13:48
author: edward
categories: blog PowerShell, Scripting
tags: WMI
slug: powershell-get-wmiobject-vs-get-ciminstance
status: published
---

A recent post on the [Hey, Scripting Guy! blog](http://blogs.technet.com/b/heyscriptingguy/ "Microsoft Technet's PowerShell Scripting Guy(s)") showed how to [use PowerShell to find a network adapter's MAC address](http://blogs.technet.com/b/heyscriptingguy/archive/2014/11/07/powertip-use-powershell-to-find-mac-address.aspx "Use PowerShell to find a MAC address"). The post provided two ways to get the information using WMI:

> Get-WmiObject win32_networkadapterconfiguration \| select description, macaddress
>
> Get-CimInstance win32_networkadapterconfiguration \| select description, macaddress

I wondered about the difference between GetWmiObject versus Get-CimInstance. Happily, while exploring older Hey, Scripting Guy! posts, I found one about [simplifying PowerShell scripts](http://blogs.technet.com/b/heyscriptingguy/archive/2014/11/02/weekend-scripter-simplify-to-troubleshoot-powershell-script.aspx "Troubleshoot and Simplify PowerShell scripts") that addressed the difference(s):

> One of the first changes I make to a script, if I can, is I change **Get-WmiObject** to **Get-CimInstance**. Since Windows PowerShell 3.0, I can use **Get-CimInstance**. It is faster and more robust, and it permits lots of cool things for retrieving data (such as using **Cim-Sessions**).

Since most of our servers here are running PowerShell 2.0 right now, it will be a while before I can routinely use Get-CimInstance instead of Get-WmiObject -- but it's something for me to keep in mind for the future.
