---
layout: page
title: "Algorithm"
permalink: /categories/algorithm/
category: algorithm
---

<ul>
  {% for cat in site.categories %}
    <li>{{ cat[0] }} - {{ cat[1].size }}</li>
  {% endfor %}
</ul>

{% assign cat = page.category %}
{% assign posts = site.categories[cat] %}

{% if posts %}
  <ul>
    {% for post in posts %}
      <li><a href="{{ post.url }}">{{ post.title }}</a> - {{ post.date | date: "%Y-%m-%d" }}</li>
    {% endfor %}
  </ul>
{% else %}
  <p>No posts found in this category.</p>
{% endif %}
