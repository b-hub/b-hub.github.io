---
layout: default
title: "Posts"
description: ""
---

{% for post in site.posts %}

## {{ post.title }}

_{{ post.date }}_

{% for category in post.categories %}
<span>{{ category }} </span>
{% endfor %}

{% for tag in post.tags %}
<span>{{ tag }} </span>
{% endfor %}

{{ post.excerpt }} 

<a href="{{ post.url }}">...read more</a>

{% endfor %}
