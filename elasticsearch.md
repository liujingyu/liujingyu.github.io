---
layout: page
title: ElasticSearch
---

{% for item in site.elasticsearch %}

<a href="{{ item.url | prepend: site.baseurl }}">
  <h2>{{ item.title }}</h2>
</a>

<p class="post-excerpt">{{ item.description | truncate: 160 }}</p>

{% endfor %}
