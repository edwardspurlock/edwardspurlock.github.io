---
title: Blog
layout: categories_projects
author: Edward Spurlock
author_profile: true
header:
 overlay_image: /assets/images/encrypted_data.png
---

## My blog, and welcome to it
I've been blogging off and on since before 2010. My original blog was a WordPress blog and focused on web design. I backed the posts up to XML files, and I'm going to try to migrate them to this blog. 

My more recent hosted blog was also WordPress, and I have posts dating back to 2014 that I've already migrated to this new website (which is a static website using Jekyll, not a dynamic WordPress site). You can see the posts from 2014-2022 listed below. Many of the posts from 2014-2018 were about Windows PowerShell. After I started working for Google Maps Platform in October 2018, most of my posts focused on Google Maps and Google in general, plus Instructional Design. 

Going forward, I intend to focus more on data analysis and data engineering. In particular, I plan to write short articles about SQL and Python, as well as continuing to have a number of posts about PowerShell and other technical subjects. I will put longer technical articles on the [Projects](/projects) page. 

I also plan to write about other (less technical) interests, including Toastmasters and social dancing.

{% assign entries_layout = page.entries_layout | default: 'list' %}
<h3 class="archive__subtitle">{{ site.data.ui-text[site.locale].recent_posts | default: "My Blog" }}</h3>
<div class="entries-{{ entries_layout }}">
{% for post in site.posts%}
    {% if post.categories contains "blog" and post.status != 'draft' %}
        {% include archive-single.html type=entries_layout %}
    {% endif %}
{% endfor %}
</div>