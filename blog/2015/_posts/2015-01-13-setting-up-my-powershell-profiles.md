---
layout: posts
title: Setting up my PowerShell profiles
date: 2015-01-13 17:32
author: edward
categories: blog PowerShell, Scripting
tags: profiles, workflow
slug: setting-up-my-powershell-profiles
status: published
---

There comes a time in every programmer's life when s/he has to strike out on his/her own, writing new code (instead of typing in examples from books / websites). That time has come now for me with regards to PowerShell.

But first, I have to set up my working environment.

Here at work, we have a common (i.e., shared) network directory on our Production resource server. There were no PowerShell utilities in the directory (probably because I think I'm the first person to do anything serious with PowerShell here, with the possible exception of the IT guys - and they don't use the Production resource server).

However, it occurred to me that that common directory (call it N:\\common\\utils, because that's not its name) would be a good place to put modules meant to be shared.

How do I tell PowerShell to look for modules there, without having to specify this every time I start PowerShell?

For now, I just:

1.  created a PS subdirectory in N:\\common\\utils (PS for PowerShell, of course)
2.  Started PowerShell on my PC and created a profile file in the \$profile directory (per Recipe 1.6 from the Windows PowerShell Cookbook):  
   New-Item -type file -force \$profile
3.  edited the profile file using Notepad.exe:  
   notepad \$profile
4.  and added a line to add the common directory to the PSModulePath environment variable:  
   \$env:PSModulePath += ";N:\\common\\utils\\PS"  
   (the leading semicolon is the profile path separator)
5.  exited notepad, saving \$profile on the way out.

 

Now, whenever I start PowerShell, the \$profile runs and adds the PS shared folder to the module search path.

To do the same thing (less step 1) for the Windows PowerShell ISE, I consulted Microsoft Technet article [How to Use Profiles in Windows PowerShell ISE](http://technet.microsoft.com/en-US/library/dd819434.aspx "How to Use Profiles in Windows PowerShell ISE - MS Technet"), which suggests wrapping the New-Item in step 2 in an **if** statement to prevent overwriting an existing profile, and using the ISE to edit the resulting profile file:

1.  (PS subdirectory already created in N:\\common\\utils)
2.  Started the PowerShell ISE and created a profile file in \$profile:  
   if (!(test-path \$profile)) {New-Item -type file -path \$profile -force}
3.  edited the profile file (using the ISE editor):  
   psEdit \$profile
4.  and added the same line to add the common directory to PSModulePath:  
   \$env:PSModulePath += ";N:\\common\\utils\\PS"
5.  then closed the ISE editor tab, saving the ISE \$profile file on the way out

 

Now I just have to figure out modules and module manifests...
