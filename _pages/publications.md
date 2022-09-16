---
layout: single
title: Publications
author_profile: true
description:
permalink: /publications/
---

During my master and PhD studies I contributed to several astrophysical publications listed in the following.

{% for post in site.posts %}
{% if post.category == 'publications'  %}
<div class="row">
	<div class="small-12 columns">
  	
		<small><i>{{ post.date | date: '%B %d, %Y' }}</i></small>
		<br>
		<a href="{{ post.url }}">{{ post.title }}</a>

		<small>
		{{ post.excerpt| truncate: 100, "..." }}
		
		
		<hr>


{% endif %}
{% endfor %}
