---
layout: page
title: ElasticSearch
---

{% for item in site.elasticsearch %}

<h2><a href="{{ item.url | prepend: site.baseurl }}">{{ item.title }}</a></h2>

<p class="post-excerpt">{{ item.description | truncate: 160 }}</p>

{% endfor %}
