---
layout: default
title: "Posts"
description: ""
---

{% for post in site.posts %}

## {{ post.title }}

_{{ page.date }}_

{% for category in post.categories %}

<span>{{ category }} </span>

{% endfor %}

{% for tag in post.tag %}

{{ tag }}

{% endfor %}

{{ post.excerpt }} 

<a href="{{ post.url }}">...read more</a>

{% endfor %}
