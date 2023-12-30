---
title: Blog
layout: categories_projects
author: Edward Spurlock
author_profile: true
---

## This is the index.md file inside the blog source directory
{% assign entries_layout = page.entries_layout | default: 'grid' %}
<h3 class="archive__subtitle">{{ site.data.ui-text[site.locale].recent_posts | default: "My Blog" }}</h3>
<div class="entries-{{ entries_layout }}">
{% for post in site.posts%}
    {% if post.categories contains "blog" %}
        {% include archive-single.html type=entries_layout %}
    {% endif %}
{% endfor %}
</div>