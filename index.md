---
layout: default
title: Home
---

# Raind Notes

- [はじめに]({{ '/docs/1_introduction.html' | relative_url }})

## Posts
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.date | date: "%Y-%m-%d" }} - {{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
