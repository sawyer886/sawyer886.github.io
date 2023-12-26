---
layout: default
title: Blog
---

<h1>最近博客</h1>

<ol>
  {% for post in site.posts %}
    <li>
      <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
      <!-- {{ post.excerpt }} -->
    </li>
  {% endfor %}
</ol>
