---
title: "논문 정리"
layout: archive
permalink: /docs/paper-summary/

author_profile: true # 게시글에 사용자의 프로필이 보이게 하기
sidebar:
    nav: "docs"
---

{% assign posts = site.categories.paper-summary %}
{% for post in posts %}
  {% include archive-single.html type=entries_layout %}
{% endfor %}