---
layout: default
title: "Blog"
description: "Projects, tips and newsletters"
permalink: /blog/
---

{% for post in site.posts %}

<div class="blog-post">

## {{ post.title }}
  
  <span class="blog-post-date">{{ post.date }}</span>

{{ post.categories | join ", " }} | {{ post.tags | join ", " }}

> {{ post.excerpt }} 

<a href="{{ post.url }}">...read more</a>
  
</div>

{% endfor %}

{% render "test" %}
