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
		<sub>{{ post.date | date: '%B %d, %Y' }}</sub>
		<a href="{{ post.url }}"><h3>{{ post.title }}</h3></a>

	  	{{ post.description }}
		{{ post.excerpt }}

	{% endif %}

	</div>
</div>
{% endfor %}
