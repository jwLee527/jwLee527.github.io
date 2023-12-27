---
title: "python"
layout: archive
permalink: /docs/python/

author_profile: true # 게시글에 사용자의 프로필이 보이게 하기
sidebar:
    nav: "docs"
---

{% assign posts = site.categories.python %}
{% for post in posts %}
  {% include archive-single.html type=entries_layout %}
{% endfor %}