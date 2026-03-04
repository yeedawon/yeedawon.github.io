---
layout: page
title: Archive
permalink: /archive/
---

{% assign posts_by_year = site.posts | group_by_exp: "p", "p.date | date: '%Y'" %}
{% for yg in posts_by_year %}
<p class="arc-year">{{ yg.name }}</p>
<ul class="arc-list">
  {% for post in yg.items %}
  <li class="arc-item">
    <span class="arc-date">{{ post.date | date: "%m.%d" }}</span>
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    {% if post.categories[0] == "springboot" %}
      <span class="badge badge-sb" style="font-size:.6rem;">SB</span>
    {% elsif post.categories[0] == "network" %}
      <span class="badge badge-net" style="font-size:.6rem;">NET</span>
    {% endif %}
  </li>
  {% endfor %}
</ul>
{% endfor %}
