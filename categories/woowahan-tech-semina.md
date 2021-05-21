---
layout: page
title: Woowahan Tech Semina
permalink: /blog/categories/woowahan-tech-semina/
---

<h5> Posts by Category : {{ page.title }} </h5>

<div class="card">
{% for post in site.categories.woowahan-tech-semina %}
 <li class="category-posts"><span>{{ post.date | date_to_string }}</span> &nbsp; <a href="{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}
</div>