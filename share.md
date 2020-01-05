---
layout: page
title: Share
---

{% for item in site.share %}

<h2><a href="{{ item.url | prepend: site.baseurl }}"> {{ item.title }}</a></h2>

<p class="post-excerpt">{{ item.description | truncate: 160 }}</p>

{% endfor %}

