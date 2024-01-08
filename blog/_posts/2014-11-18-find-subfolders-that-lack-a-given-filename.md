---
layout: posts
title: Find subfolders that lack a given file(name)
date: 2014-11-18 16:29
author: edward
categories: blog PowerShell, Scripting
tags: files
slug: find-subfolders-that-lack-a-given-filename
status: published
---

Our processing automation at work creates a number of files during processing. One way we can tell when the automation hasn't completed successfully is when the processed files directory has been created, but the files that are created at the end of processing are missing from the directory.

Here's a PowerShell script fragment to identify subfolders (two levels down) that lack a given file (target.txt in this example) :

> PS \> Get-ChildItem \|  
> \>\>   ForEach-Object {  
> \>\>     Set-Location \$\_  
> \>\>     Get-ChildItem \|  
> \>\>       Where-Object {!(Test-Path \$\_\\target.txt)}  
> \>\>     Set-Location ..  
> \>\> }  
> \>\>

And here's a one-liner version of the above:

> PS \> ls \| %{ cd \$\_ ; ls \| ?{!(test-path \$\_\\target.txt)} ; cd ..}

It's not perfect - if it encounters a file (rather than a subdirectory) one level down, it attempts to Set-Location to that filename and throws an error (because you can't Set-Location to a file, only a folder). However, it seems to find all the folders missing the target.txt file before it attempts to Set-Location to that file.
