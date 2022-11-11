---
layout: search
title: "Search"
permalink: /search/
main_nav: true
---

{% for cat in site.descriptions %}
  <h2 id="{{cat}}">{{ cat }}</h2>
  <ul class="posts-list">
  {% for post in site.categories[cat] %}
    <li>
      <strong>
        <a href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
      </strong>
    </li>
  {% endfor %}
  </ul>
  {% if forloop.last == false %}<hr>{% endif %}
{% endfor %}
<br>
