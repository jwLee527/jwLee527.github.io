---
title: "이것저것 정리"
layout: archive
permalink: /docs/others-summary/

author_profile: true # 게시글에 사용자의 프로필이 보이게 하기
sidebar:
    nav: "docs"
---

{% assign posts = site.categories.others-summary %}
{% for post in posts %}
  {% include archive-single.html type=entries_layout %}
{% endfor %}