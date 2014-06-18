---
layout: page
title: Categories
permalink: /Category/
---

{% for category in site.categories %}
  <li><a name="{{ category | first }}">{{ category | first }}</a>
    {% for posts in category %}
    <ul>
      {% for post in posts %}
      	{% if post.title %}
      		<li><a href="{{ post.url }}">{{ post.title }}</a></li>
      	{% endif %}
      {% endfor %}
    </ul>
    {% endfor %}
  </li>
{% endfor %}



