---
layout: page
title: Blog
permalink: /blog/
---

# Blog

Longer technical posts and deep dives into software engineering topics.

{% assign sorted_blog = site.blog | sort: 'date' | reverse %}
{% if sorted_blog.size > 0 %}
{% for post in sorted_blog %}
## [{{ post.title }}]({{ post.url }})
*{{ post.date | date: "%B %d, %Y" }}*

{{ post.excerpt | strip_html }}

[Read more →]({{ post.url }})

**Tags:** {% for tag in post.tags %}#{{ tag }} {% endfor %}

---
{% endfor %}
{% else %}
## Coming Soon

I'm working on some longer technical posts. In the meantime, check out my quick learnings in the [TIL section](/til/) or my [Substack](https://sreenathmmenon.substack.com/).

## What's Coming

- Building a Multi-LLM Interface: Design Decisions and Trade-offs
- Infrastructure AI: Making Complex Systems More Accessible
- Lessons from 13+ Years of Production Code

*Subscribe to my [Substack](https://sreenathmmenon.substack.com/) for updates when new posts are published.*
{% endif %}