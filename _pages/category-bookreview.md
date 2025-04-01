---
layout: page
title: "Bookreview"
permalink: /categories/bookreview/
category: bookreview
---

{% assign cat = page.category | downcase %}
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
