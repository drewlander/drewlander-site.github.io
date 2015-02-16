---
layout: default
---
<ul>
  {% for post in site.posts limit: 5 %}
    <li><a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
{% sidebar %}
  Stuff you want in your sidebar
{% endsidebar %}
