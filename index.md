---
layout: page
---

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>


{% for post in site.posts limit:5 %}
  <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
  <div>
  {{post.content}}
  </div>
{% endfor %}
