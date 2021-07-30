---
layout: default
title: "Blog"
description: "Projects, tips and newsletters"
permalink: /blog/
---

{% for post in site.posts %}

<div class="blog-post">

  <h2 class="blog-post-title">
    <a href="{{ post.url }}">{{ post.title }}</a>
    <div class="blog-post-date">{{ post.date }}</div>
  </h2>

  <div class="blog-post-tags">
    {% for category in post.categories %}<a class="blog-post-category">{{ category }}</a>{% endfor %}
    {% for tag in post.tags %}<a class="blog-post-tag">{{ tag }}</a>{% endfor %}
  </div>
    
  <div class="blog-post-excerpt">{{ post.excerpt }}</div>
  
</div>

{% endfor %}
