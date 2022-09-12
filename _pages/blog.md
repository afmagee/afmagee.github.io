---
layout: single
title: ""
permalink: /blog/
---

{% for blog in site.posts %}
  <div class="blog">
    <h2><a href="{{ blog.url }}">{{ blog.title }}</a></h2>
  </div>
{% endfor %}

<!-- https://learn.cloudcannon.com/jekyll/introduction-to-jekyll-collections/ -->

<!-- This generates a page with links to all md files in _blog-->
<!-- {% for blog in site.blog %}
  <div class="blog">
    <h2><a href="{{ blog.url }}">{{ blog.title }}</a></h2>
  </div>
{% endfor %} -->

<!-- This shoves everything in _blog into one big old page-->
<!-- {% for blog in site.blog %}
  {{blog.content}}
{% endfor %} -->
