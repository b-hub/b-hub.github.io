---
layout: default
title: "Blog"
description: "Projects, tips and newsletters"
permalink: /blog/
---

{% for post in site.posts %}

<a class="blog-post" href="{{ post.url }}">

## {{ post.title }}
  
  <span class="blog-post-date">{{ post.date }}</span>

  {% for category in post.categories %}<span class="blog-post-category">{{ category }}</span>{% endfor %}
  {% for tag in post.tags %}<span class="blog-post-tag">{{ tag }}</span>{% endfor %}
  
> {{ post.excerpt }} 
  
</a>

{% endfor %}
