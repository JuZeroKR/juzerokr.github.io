---
title: "Daily"
layout: category
category:  daily
permalink: /categories/daily/
author_profile: true
sidebar_page: true
---

{% assign posts = site.categories.Daily %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}