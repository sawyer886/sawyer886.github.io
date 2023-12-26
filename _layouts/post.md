---
layout: default
---

<h1>{{ page.title }}</h1>

<p>
  {{ page.date | date: "%Y年%m月%d日"  }}
  {% assign catalog = site.catalogs | where: 'short_name', page.catalog | first %}
  {% if catalog %}
    - <a href="{{ catalog.url }}">{{ catalog.name }}</a>
  {% endif %}
</p>

{{ content }}
