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
toc: true
toc_label: Contents
toc_sticky: true
---
## Summary:
I adapted Google's open source C# code for signing Google Maps Static or Street View Static URLs into a PowerShell module.<!--more--> I used PowerShell's Add-Type cmdlet to make the C# object and method available as a PowerShell function, and built a script function to sign URLs passed in a PowerShell pipeline. I added a Test-URL function to validate URLs. I also wrote comment-based help comments in the script, and created a README file to explain how to install and use the module. Finally, I created a pull request to see if my script could be added to the Google Maps open source Github repository. 

## Contents:
* <a href="#background">Background</a>
* <a href="#creating_a_module">Creating a PowerShell module</a>
    * <a href="#proof_of_concept">Proof of concept</a>
    * <a href="#first_version">First version of the module</a>
    * <a href="#code_review_script_enhancement">Code review and script enhancements</a>
    * <a href="#pull_request">Pull request and aftermath</a>

<h2 id="background">Background:</h2>
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

Instead of signing URLs one at a time, you can copy the secret key from the Keys & Credentials page, and use it in code on your web server to sign each map or Street View URL when the page is built and served. The Google Maps documentation has [sample code to sign Maps URLs](https://developers.google.com/maps/documentation/maps-static/digital-signature#sample-code-for-url-signing) in several server-side languages, including Python, Java, NodeJS, and C#. There is also an [open source Github repository](https://googlemaps.github.io/url-signing/index.html) with sample code in PHP, Perl, Ruby, VB.NET, Objective-C, and Go.

<h2 id="creating_a_module">Creating a PowerShell module</h2>
<h3 id="proof_of_concept">Proof of concept</h3>
While working as a contractor supporting Google Maps Platform, I learned how to use the Maps Static and Street View Static APIs, including creating signed URLs. As an exercise, I decided to try writing a PowerShell module to sign Maps or Street View Static URLs using the secret key from a Google Cloud Platform project. 

I did a bit of research to discover if PowerShell had a built-in way to create digital signatures for text strings, but I was not able to find any PowerShell cmdlets with this functionality.

I had the idea that C# code could be used in PowerShell, so I did some more research and found the [Add-Type](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/add-type) cmdlet. This cmdlet takes .NET code and makes its classes and methods available as functions in PowerShell scripts. 

I copied the C# ['GoogleSignedUrl' code from the Google documentation](https://developers.google.com/maps/documentation/maps-static/digital-signature#c), pasted it in a [new script](https://github.com/edwardspurlock/URLsigner/blob/main/URLsigner.psm1#L35), and assigned it to a PowerShell variable:
{% highlight powershell %}
PS > $code = @"
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
PS >  
{% endhighlight %}
  
I then used the [Add-Type](https://github.com/edwardspurlock/URLsigner/blob/main/URLsigner.psm1#L67) cmdlet to import the <code>_SignUrl</code> class and the <code>Sign</code> function into PowerShell, then used <code>New-Object</code> to instantiate the new class:
{% highlight powershell %}
PS > Add-Type -TypeDefinition $code
PS > $signer = New-Object _SignUrl
{% endhighlight %}

I tested the new function by: 
* creating a valid Maps Static URL in PowerShell
* copying the Secret Key from my Google Cloud Project and pasting it into a PowerShell variable
* signing the URL using the instantiated _SignUrl object
* comparing the signature to one created using the "Sign a URL now" tool in my Google Cloud Project
{% highlight powershell %}
PS > $unsigned_url = "https://maps.googleapis.com/maps/api/staticmap?center=eiffel+tower,paris,france&zoom=12&size=200x200&key=AIzaSyA6x050-z4FE_0aYf4E2h64VBgP1S13_Jg"
PS > $secret_key = "Nn***********************Mw="
PS > $signer.Sign($unsigned_url, $secret_key)
https://maps.googleapis.com/maps/api/staticmap?center=eiffel+tower,paris,france&zoom=12&size=200x200&key=AIzaSyA6x050-z4FE_0aYf4E2h64VBgP1S13_Jg&signature=g1sct1Y5-W6u_8KXuPQmx5XjtKE=
{% endhighlight %}

The signature parameter created by <code>$signer.Sign()</code>  
(<code>&signature=g1sct1Y5-W6u_8KXuPQmx5XjtKE=</code>)  
is identical to one created by the "Sign a URL now" tool:
![screenshot showing signed Google Maps Static URL](/assets/images/Signed_URL.png)

<h3 id="first_version">First version of the module</h3>
To turn this code into a module, I started by creating a function named <code>signStaticUrl</code>, with a PowerShell <code>begin ... process ... end</code> framework. I made the script's $url argument mandatory and able to accept value(s) from the PowerShell pipeline.  

I put the instantiation of the _SignUrl function into the <code>begin</code> block, so it would run only once when processing multiple URLs in a pipeline.  

In the <code>process</code> block, I used a regex to remove any existing signature on the input $url, then used the $signer.Sign() method to sign the URL and pass it to the script output:
{% highlight powershell %}
  $url = $url -replace '&signature=.*$', ''
  $script:signer.Sign($url, $secretKey)
{% endhighlight %}

I added two aliases for the function:
{% highlight powershell %}
  Set-Alias -Name Protect-StaticUrl -Value signStaticUrl
  Set-Alias -Name Protect-Url -Value signStaticUrl
{% endhighlight %}

To prevent users from having to copy the secret key from their Google Cloud project each time the script was used, I added a function <code>Set-SecretKey</code> and added comments to show where to paste the key, as well as specifying the key each time the script was run if desired.

<h3 id="code_review_script_enhancement">Code review and script enhancements</h3>
Once I had a script that could be used as a standalone command line function or as part of a PowerShell pipeline, I contacted the person in charge of the [Google Maps open source libraries](https://googlemaps.github.io/libraries) to see if I could get my PowerShell script added to the [URL Signing Samples](https://googlemaps.github.io/url-signing/index.html) repository. They told me the first step would be for me to get my code reviewed by a Google Maps engineer. 

I reached out to the Google engineers that I worked with in my role as a contractor for Google Maps Platform support. One of the engineers volunteered to review my code, and verified that the script worked correctly. He suggested that I add code to verify that the input URLs were valid. This seemed like a great idea, so I added a new function, <code>Test-StaticUrl</code>, which uses regular expressions to:
* verify that the input URL contains a 39-character Google Cloud API key
* verify that the first part of the URL has the correct domain and path
* verify that the URL contains a **size** parameter (required for both Maps Static and Street View Static APIs)
* verify that the URL contains either
    * **zoom** and **center** parameters or a **marker** parameter for the Maps Static API, or
    * a **location** or a **pano** parameter for the Street View Static API

I also added [comment-based help](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_comment_based_help) to the functions and to the script as a whole. I created a directory in my copy of the repo and moved the script into the directory to allow it to be loaded as a PowerShell module. Finally, I wrote a [README.md](https://github.com/edwardspurlock/URLsigner?tab=readme-ov-file#urlsigner) with screenshots to explain how to install and use the module. 

<h3 id="pull_request">Pull request and aftermath</h3>
I had the Google engineer sign off on the new <code>Test-StaticUrl</code>, comment-based help, and README. I then created a pull request against the URL Signing Samples repo on Github to have my submission evaluated for inclusion in the repo.

After evaluation, the person in charge got back to me to request that I submit my code as a single script (similar to the existing code samples in the repo), rather than as a PowerShell module directory with separate README. 

I started thinking about what I should do to simplify my code and resubmit. However, we became very busy at work, and I put the project away for a few weeks. About the time I started thinking about resuming work on the project, Covid hit, and what with dealing with remote work, I never got back to it. 

Shortly after that, Google archived their open source library on Github and made it read-only, so I don't anticipate getting back to this project again. However, the PowerShell module is complete and self-contained as it is, and still works with the Maps Static and Street View Static APIs.

