---
layout: default
title: "Blog"
description: "Projects, tips and newsletters"
permalink: /blog/
---

{% for post in site.posts %}

## {{ post.title }}

_{{ post.date }}_

{{ post.categories | join ", " }} | {{ post.tags | join ", " }}

> {{ post.excerpt }} 

<a href="{{ post.url }}">...read more</a>

{% endfor %}
