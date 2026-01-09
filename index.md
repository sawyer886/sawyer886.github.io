---
layout: default
title: Home
---
English | [中文](/index-zh.html)


# Sawyer | Senior Big Data Engineer

**10+ Years Experience** in Big Data & Data Engineering

Currently seeking **remote opportunities** in Data Engineering, Full Stack Development, and AI/ML roles.

[View Resume](/about/) | [Read Blog](/blog.html) | [GitHub](https://github.com/sawyer886)

---

## Core Expertise
- 🔧 **Big Data**: Spark, Hadoop, Flink, HBase
- 💻 **Full Stack**: Java, Python, Go, JavaScript
- ⛓️ **Web3**: Solidity, Rust
- 🤖 **AI Tools**: Cursor, GitHub Copilot

## Latest Articles
{% for post in site.posts limit:3 %}
### [{{ post.title }}]({{ post.url }})
*{{ post.date | date: "%B %d, %Y" }}* - {{ post.categories | join: ", " }}
{% endfor %}

[View All Posts →](/blog.html)
