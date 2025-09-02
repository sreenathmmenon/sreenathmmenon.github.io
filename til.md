---
layout: page
title: Today I Learned
permalink: /til/
---

Quick notes on things I discover while coding. Inspired by [Hashrocket's TIL](https://til.hashrocket.com/).

{% assign sorted_til = site.til | sort: 'date' | reverse %}
{% for post in sorted_til %}
## [{{ post.title }}]({{ post.url }})
*{{ post.date | date: "%B %Y" }}*

{{ post.content | strip_html | truncate: 150 }}

[Read more →]({{ post.url }})

**Tags:** {% for tag in post.tags %}#{{ tag }} {% endfor %}

---
{% endfor %}

*More TIL posts coming as I learn new things...*