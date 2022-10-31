---
layout: single
title: Publications
author_profile: true
description:
permalink: /publications/
---

During my master and PhD studies I contributed to several astrophysical publications. The main ones are listed here.

{% for post in site.posts %}
{% if post.category == 'publications'  %}
<div class="row">
	<div class="small-12 columns">
  	
		<small><i>{{ post.date | date: '%B %d, %Y' }}</i></small>
		<br>
		<a href="{{ post.url }}">{{ post.title }}</a>
    		<br>
		<i><small>Authors: {{ post.authors }}</small></i>
		<br>
		<small>{{ post.preview_text }}
		<br>
		<div style="text-align: right">
		<p style="color:green">Citations: {{ post.citations }}</p>
		</div>
		</small>
		<hr>
		

{% endif %}
{% endfor %}
