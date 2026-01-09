---
layout: default
title: 首页
---

[English](/index.html) | 中文

# Sawyer | 高级大数据工程师

**10年以上**大数据与数据工程经验

目前寻求**远程工作机会**，涵盖数据工程、全栈开发和AI/ML领域。

[查看简历](/about/) | [阅读博客](/blog.html) | [GitHub](https://github.com/sawyer886)

---

## 核心专长

- 🔧 **大数据**: Spark, Hadoop, Flink, HBase
- 💻 **全栈开发**: Java, Python, Go, JavaScript
- 🌐 **Web3**: Solidity, Rust
- 🤖 **AI工具**: Cursor, GitHub Copilot

## 最新文章

{% for post in site.posts limit:3 %}
### [{{ post.title }}]({{ post.url }})
*{{ post.date | date: "%Y年%m月%d日" }}* - {{ post.categories | join: ", " }} {% endfor %}

[查看所有文章 →](/blog.html)
