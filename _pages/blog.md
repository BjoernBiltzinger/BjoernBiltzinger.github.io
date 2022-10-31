---
layout: single
title: Blog
description:
author_profile: true
permalink: /blog/
---

A few general ideas I want to share.

{% for post in site.posts %}
{% if post.category == 'blog'  %}
<div class="row">
	<div class="small-12 columns">
  	
		<small><i>{{ post.date | date: '%B %d, %Y' }}</i></small>
		<br>
		<a href="{{ post.url }}">{{ post.title }}</a>
    	<br>
		<small>
        {{ post.preview_text }}
		</small>
		<hr>
		

{% endif %}
{% endfor %}
