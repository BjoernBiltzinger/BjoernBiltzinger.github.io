---
layout: single
title: Publications
author_profile: true
description:
permalink: /publications/
---

During my master and PhD studies I contributed to several astrophysical publications listed in the following.

{% for post in site.posts %}
<div class="row">
	<div class="small-12 columns">
  	
	 {% if post.category == 'publications'  %}	
		<i>{{ post.date | date: '%B %d, %Y' }}</i>
		<a href="{{ post.url }}">{{ post.title }}</a>

		{{ post.excerpt| truncate: 100, "..." }}
		
		<hr>
	{% endif %}

	</div>
</div>
{% endfor %}
