---
layout: archive
permalink: /docs/blog/
title: "blog"

author_profile: true
sidebar:
  nav: "docs"
---

{% assign posts = site.categories.blog %}
{% for post in posts %}
  {% include custom-archive-single.html type=entries_layout %}
{% endfor %}