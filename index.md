---
layout: archive
permalink: /
title: "Latest posts"
image:
  feature: fernando-de-noronha.jpg
id: home
---

<div class="tiles">
{% for post in site.posts %}
	{% include post-grid.html %}
{% endfor %}
</div><!-- /.tiles -->
