---
layout: page
title: Welcome to Ben's Blog
tagline: Supporting tagline
---
{% include JB/setup %}

{% for post in site.posts offset:0 limit: 10 %}

# [{{ post.title }}]({{ post.url }})

{{ post.summary }}

Date: {{ post.date }} 
Tags: {% for tag in post.tags %}
[{{ tag }}](/tags/{{ tag }})
{% if forloop.last != true %} {% endif %}
{% endfor %}		            

{% endfor %}
