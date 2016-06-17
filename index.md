---
layout: page
---

{% for post in site.posts limit:5 %}
  <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
  <div>
  {{post.content}}
  </div>
{% endfor %}
