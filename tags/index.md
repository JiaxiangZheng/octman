---
title: 标签
layout: page
---

<div id='tag_cloud'>
{% for tag in site.tags %}
<a href="#{{ tag[0] }}" title="{{ tag[0] }}" rel="{{ tag[1].size }}">{{ tag[0] }}<sup>{{ tag[1].size }}</sup></a>
{% endfor %}
</div>

<ul class="listing">
  {% for tag in site.tags %}
    <h2 class="listing-seperator" id="{{ tag[0] }}">{{ tag[0] }}</h2>
  {% for post in tag[1] %}
    <li class="listing-item">
    <time datetime="{{ post.date | date:"%Y-%m-%d" }}">{{ post.date | date:"%Y-%m-%d" }}</time>
    <a href="{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a>
    </li>
  {% endfor %}
  {% endfor %}
</ul>

