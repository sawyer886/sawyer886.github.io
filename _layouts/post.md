---
layout: default
---

<h1>{{ page.title }}</h1>

<p>
  {{ page.date | date: "%Y年%m月%d日"  }}
  {% assign author = site.authors | where: 'short_name', page.author | first %}
  {% if author %}
    - <a href="{{ author.url }}">{{ author.name }}</a>
  {% endif %}
</p>

{{ content }}
