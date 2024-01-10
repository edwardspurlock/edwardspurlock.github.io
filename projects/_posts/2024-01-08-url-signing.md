---
layout: posts
title:  "A PowerShell module to sign Google Maps URLs"
date:  2023-07-21 17:42:14 +0000
categories: PowerShell
highlight_home: true
tags: 
description: I adapted Google's open source C# code to sign Static Maps URLs into a PowerShell module
header:
 overlay_image: /assets/images/encrypted_data.png
 teaser: /assets/images/PowerShell.svg
---
## Summary:
I adapted Google's open source C# code for signing sign Static Maps URLs into a PowerShell module.<!--more--> 

## Background:
Google Maps Platform makes a number of services available through REST APIs. Two of the simplest APIs are the Maps Static and Street View Static services. These services allow you to add a static image of a map or Street View scene to a web page in an \<img\> tag. For example, you can generate a map showing the Eiffel Tower using:
{% highlight html %}
https://maps.googleapis.com/maps/api/staticmap?center=eiffel+tower,paris,france&zoom=12&size=200x200&key=AIza...
{% endhighlight %}
<!--
![Eiffel Tower, Paris, France](https://maps.googleapis.com/maps/api/staticmap?center=eiffel+tower,paris,france&zoom=12&size=200x200&key=AIzaSyA6x050-z4FE_0aYf4E2h64VBgP1S13_Jg)
-->
![temporary placeholder for Google Map while developing](https://placehold.co/200x200)

When a Google Cloud API key is used in a URL like this, the API key is embedded in the HTML source of the page, and can be viewed using the "View Source" or "Inspect" browser functions. An unprotected API key could be copied from the HTML source and used to make API requests that would be charged to the API key's owner.

To protect an API key used for Maps Static URLs, you can use a number of strategies:
* Create a dedicated API key with **API restriction** to limit the key to the Maps Static and/or Street View Static APIs
* Configure **Application restriction** in the key setting to limit the key to your website(s)
* Set a **quota limit** on the Maps Static / Street View Static APIs to limit the number of API accesses per day
* Add a **digital signature** to each URL

You can sign a URL by pasting the entire unsigned URL into the "Sign a URL now" tool on the Google Maps Platform Keys & Credentials page:
![screenshot of Google Maps Platform Keys & Credentials page](/assets/images/gmp_keys_credentials.png)  

The signed URL can then be copied from the tool and pasted into an \<img\> tag in your site's HTML.

This process works well enough for one or two URLs, but if your site generates dozens or hundreds of map or Street View images from addresses in a database, pasting each unsigned URL into the "Sign a URL now" tool, copying the signed URL and pasting it into the HTML source of each web page would be tedious and prone to errors.

Instead of signing URLs one at a time, you can copy the **Current secret** from the Keys & Credentials page, and use it in code on your web server to sign each map or Street View URL when the page is built and served. The Google Maps documentation has [sample code to sign Maps URLs](https://developers.google.com/maps/documentation/maps-static/digital-signature#sample-code-for-url-signing) in several server-side languages, including Python, Java, NodeJS, and C#. There is also an [open source Github repository](https://googlemaps.github.io/url-signing/index.html) with sample code in PHP, Perl, Ruby, VB.NET, Objective-C, and Go.

## Actions:
As an exercise, I decided to try writing a PowerShell module to sign Maps or Street View Static URLs using the secret key from a Google Cloud Platform project. 

I did a bit of research to discover if PowerShell had a built-in way to create digital signatures for text strings, but I was not able to find any PowerShell cmdlets with this functionality.

I had the idea that C# code could be used in PowerShell, so I did some more research and found the [Add-Type](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/add-type) cmdlet. This cmdlet takes .NET code and makes its classes and methods available as functions in PowerShell scripts. 

I copied the C# ['GoogleSignedUrl' code from the Google documentation](https://developers.google.com/maps/documentation/maps-static/digital-signature#c), pasted it in a [new script](https://github.com/edwardspurlock/URLsigner/blob/main/URLsigner.psm1#L35), and assigned it to a PowerShell variable:
{% highlight powershell %}
$code = @"
using System;
using System.Collections.Generic;
...
public class _SignUrl
{
    public string Sign(string url, string secretKey ) {
      ASCIIEncoding encoding = new ASCIIEncoding();
      // converting key to bytes will throw an exception,
...
      // Add the signature to the existing URI.
      return uri.Scheme+"://"+uri.Host+uri.LocalPath + uri.Query +"&signature=" + signature;
   }
}
"@
{% endhighlight %}
  
I then used the [Add-Type](https://github.com/edwardspurlock/URLsigner/blob/main/URLsigner.psm1#L67) cmdlet to import the <code>_SignUrl</code> class and the <code>Sign</code> function into PowerShell:
{% highlight powershell %}
# add the .NET class _SignUrl to this PowerShell session
Add-Type -TypeDefinition $code
{% endhighlight %}

