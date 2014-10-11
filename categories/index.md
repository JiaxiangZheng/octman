---
title: 分类
layout: page
---

<ul class="listing">
{% for cat in site.categories %}
  <h2 class="listing-seperator" id="{{ cat[0] }}">{{ cat[0] }}</h2>
{% for post in cat[1] %}
  <li class="listing-item">
  <time datetime="{{ post.date | date:"%Y-%m-%d" }}">{{ post.date | date:"%Y-%m-%d" }}</time>
  <a href="{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a>
  </li>
{% endfor %}
{% endfor %}
</ul>
