---
layout: posts
title:  "Reading XML in PowerShell"
date:   2024-02-01 22:25:00 -0600
categories: PowerShell
highlight_home: true
description: PowerShell makes it easy to read XML files in a structured way
header:
 overlay_image: /assets/images/encrypted_data.png
 teaser: /assets/images/PowerShell.svg
excerpt: Need to convert XML files into another format? <br>Don't convert - read the properties directly using PowerShell
status: published
---
## Migrating posts from WordPress to Jekyll
From 2011 to 2013, I created a blog using WordPress. In December 2013, I decided to migrate my blog to a new hosting provider. I backed up the posts on the old blog to XML files, and downloaded the .ZIP archive to my local hard drive. I set up WordPress with my new hosting provider and started writing new posts, figuring I'd import the posts from the old blog at a later time.  

Except I never did. The old posts have been sitting in folders in my local hard drive, untouched for ten years.

Now I've migrated my blog yet again, this time away from WordPress to a static web site generated using the Jekyll static site generator framework. I downloaded the posts from the more recent WordPress blog and converted the posts into Markdown format when I was experimenting with a different static site generator. I wrote a short PowerShell command line function to convert the posts from the other static site framework to the Markdown format that Jekyll uses. You can see them in the list of historical posts on the blog landing page.

Now that I have this new static site more or less ready to go, I decided to take a look at converting the XML files from the old blog into Markdown format. I didn't want to go through the process of converting the posts to Markdown using the other static site framework only to rework them in Jekyll. 

The tools available in the [jekyll-import](https://import.jekyllrb.com/docs/home/) site include a tool to migrate WordPress posts and pages from a live site (which probably would have been easier to use on my newer site), but at first glance, they don't appear to include an option to import XML files extracted from a WordPress backup file.

Before I started doing in-depth research to see if one of the existing converters could be used, I decided to try reading the XML files using PowerShell. I figured that even if I had to use one of the jekyll-import options in the end, the PowerShell research might come in handy in the future.

## Reading XML files the PowerShell way
At first, I tried using the PowerShell <code>Select-Xml</code> cmdlet, but when I read the a file using the standard PowerShell <code>Get-Content</code> command, the file was read as an unstructured collection of strings, and raised a multitude of error messages when I tried to pass the file into <code>Select-Xml</code>.

It turns out there's an easier way. If you read the file into a PowerShell variable using <code>Get-Content</code>, you can cast the receiving variable to make it a **structured** XML object, instead of an unstructured collection of strings:
```
PS > [xml]$old_blog_post = Get-Content .\my-first-blog-post.xml
```
The **[xml]** makes the <code>$old_blog_post</code> variable an XML object, and the properties of the variable can be read using simple dot notation:
```
PS > $old_blog_post.title
Hello World!
PS > $old_blog_post.author
edward
PS > $old_blog_post.pubDate
2011-07-16 22:37:00
```

Of course, I can also use the PowerShell pipeline to read multiple files, extract the information I want from each one, wrap each post's properties in double-quotes, and write them out as lines that could be saved as a CSV file:
```
PS > Get-ChildItem *.xml | ForEach-Object {
    [xml]$post = Get-Content $_;
    $title = $post.post.title;
    $pubDate = $post.post.pubDate;
    '"'+$pubDate+'","'+$title+'"'
}
"2011-08-29 09:52:00","This is a test post"
"2011-03-24 18:42:00","Smashing Magazine: How to Choose a Typeface"
"2011-07-15 21:05:00","Design Inspiration Countdown #7 - One Bit Increment"
"2011-06-29 10:50:00","Copywriting for Toastmasters"
...
```

There is a CSV file importer in the jekyll-import list of tools, but I think it will be almost as easy (and better practice for my PowerShell skills) to extend this quick and dirty PowerShell command line to generate complete blog posts. Either way, I should have my old blog posts back online after 10 years. What will I find that I've forgotten?