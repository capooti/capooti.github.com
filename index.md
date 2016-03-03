---
layout: page
title: Thinking in GIS
tagline:
---
{% include JB/setup %}

{% for post in site.posts %}
  * {{ post.date | date_to_string }}, <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}

    > {{ post.content| strip_html | truncatewords: 75 }}

{% endfor %}
