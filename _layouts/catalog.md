---
layout: default
---
<h1>{{ page.name }}</h1>
<!-- <h2>{{ page.position }}</h2> -->

{{ content }}

<h2>分类博客列表</h2>
<ul>
  {% assign filtered_posts = site.posts | where: 'catalog', page.short_name %}
  {% for post in filtered_posts %}
    <li><a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>