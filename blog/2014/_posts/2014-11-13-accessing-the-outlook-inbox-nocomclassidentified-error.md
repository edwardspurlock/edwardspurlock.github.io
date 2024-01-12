---
layout: posts
title: Accessing the Outlook Inbox - 'NoCOMClassIdentified' error
date: 2014-11-13 15:29
author: edward
categories: blog PowerShell, Scripting
tags: COM, Exchange, Office, Outlook, PowerShell, VBA
slug: accessing-the-outlook-inbox-nocomclassidentified-error
status: published
---

We get a lot of email where I work. A metric butt-ton.

One of the things I'd like to be able to do is access that email programmatically, in order to extract production processing reports for further analysis.

As a first step, I went looking for a way to access the Inbox using PowerShell. I found a post on Microsoft's [Hey, Scripting Guy blog](http://blogs.technet.com/b/heyscriptingguy/): Use [PowerShell to Data Mine Your Outlook Inbox](http://blogs.technet.com/b/heyscriptingguy/archive/2011/05/26/use-powershell-to-data-mine-your-outlook-inbox.aspx).

There was just one problem - when I tried the steps to access the Inbox, I got a major error:

> PS C:\\Windows\\system32\> add-type -assembly "Microsoft.Office.Interop.Outlook"  
> PS C:\\Windows\\system32\> \$olFolders = "Microsoft.Office.Interop.Outlook.olDefaultFolders" -as \[type\]  
> PS C:\\Windows\\system32\> \$outlook = new-object -comobject outlook.application  
> [new-object : Retrieving the COM class factory for component with CLSID {0006F03A-0000-0000-C000-000000000046} failed]{style="color: #ff0000;"} [ due to the following error: 80080005 Server execution failed (Exception from HRESULT: 0x80080005]{style="color: #ff0000;"} [ (CO_E_SERVER_EXEC_FAILURE)).]{style="color: #ff0000;"} [ At line:1 char:12]{style="color: #ff0000;"} [ + \$outlook = new-object -comobject outlook.application]{style="color: #ff0000;"} [ + \~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~]{style="color: #ff0000;"} [ + CategoryInfo : ResourceUnavailable: (:) \[New-Object\], COMException]{style="color: #ff0000;"} [ + FullyQualifiedErrorId : NoCOMClassIdentified,Microsoft.PowerShell.Commands.NewObjectCommand]{style="color: #ff0000;"}

I searched for "NoCOMClassIdentified" and found a couple of references. I finally found one (I don't have the link, alas) that helped me realize the problem was that I was running it from a PowerShell session that I'd opened as Administrator (for teh POWER!!!).

When I opened PowerShell as myself, I was able to complete the sequence to access the 60,000+ messages in my Inbox (did I mention we get a LOT of email here?), and use a quick pipe to Where-Object to extract just the emails I sent to myself:

> PS P:\\\> Add-Type -Assembly "Microsoft.Office.Interop.Outlook"  
> PS P:\\\> \$olfolders = "Microsoft.Office.Interop.Outlook.olDefaultFolders" -as \[type\]  
> PS P:\\\> \$outlook = new-object -comobject outlook.application  
> PS P:\\\> \$namespace = \$outlook.GetNameSpace("MAPI")  
> PS P:\\\> \$folder = \$namespace.getDefaultFolder(\$olFolders::olFolderInBox)  
> PS P:\\\> \$folder.items \| Select-Object -Property Subject, ReceivedTime, Importance, SenderName \|  
> \>\>  Where-Object {\$\_.SenderName -like "Spurlock\*"}Subject ReceivedTime Importance SenderName  
> ------- ------------ ---------- ----------  
> How to send an email from ... 10/23/2014 7:01:29 PM 1 Spurlock, Edward (Austin)  
> RE: How to send an email f... 10/23/2014 7:06:15 PM 1 Spurlock, Edward (Austin)  
> PowerShell script to updat... 10/28/2014 6:17:21 PM 1 Spurlock, Edward (Austin)  
> RE: PowerShell script to u... 10/28/2014 6:25:15 PM 1 Spurlock, Edward (Austin)

Accessing email through Outlook works, but according to [Bill Long's Exchange Blog](http://blogs.technet.com/b/bill_long/), [using the Exchange Web Services (EWS) Managed API for PowerShell scripting](http://blogs.technet.com/b/bill_long/archive/2010/04/06/accessing-the-information-store-from-powershell.aspx) offers a number of advantages. For one, you can access Exchange data from a system that does not have Outlook installed. Another advantage is being able to do "ranged retrievals" - accessing a subset of the data on each retrieval, rather than having to pull everything down at once and sort it out (as I did with Where-Object).
