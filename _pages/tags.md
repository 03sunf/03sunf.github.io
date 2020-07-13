---
layout: page
title: Tags
permalink: /tags
---

{% for tag in site.data.tags %}
  <span class="tag" data-tag="{{tag}}">
    {{ site.data.format[tag] }}
  </span>
{% endfor %}
