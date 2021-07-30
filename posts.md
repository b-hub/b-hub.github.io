---
layout: default
title: "Posts"
description: ""
---

{% for post in site.posts %}

## {{ post.title }}

{{ post.categories }}

{{ post.tags }}

{{ post.excerpt }} <a href="{{ post.url }}">...read more</a>

{% endfor %}
