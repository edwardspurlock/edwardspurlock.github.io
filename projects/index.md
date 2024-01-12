---
title: Projects
layout: categories_projects
author: Edward Spurlock
author_profile: true
permalink: /projects/
header:
 overlay_image: /assets/images/encrypted_data.png
---

## This is the index.md file inside the projects source directory
{% assign entries_layout = page.entries_layout | default: 'grid' %}
<div class="entries-{{ entries_layout }}">
  {% for post in site.posts %}
   {% if post.categories contains "projects" %}
    {% include archive-single.html type=entries_layout %}
   {% endif %}
  {% endfor %}
</div>