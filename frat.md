---
layout: page
---

{% for post in site.categories.frat %}
 <li><a href="{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}

