---
title: "Posts by Tag"
permalink: /tags/
layout: tags
author_profile: true
---
{% include group-by-array collection=site.posts field="tags" %}
<ul>
  {% for tag in site.tags %}
    <span>
      <a href="#{{ tag | first }}">
        {{ tag | first }}
      </a> &nbsp;&nbsp;&nbsp;
    </span>
  {% endfor %}
</ul>