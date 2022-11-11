---
layout: page
title: "모든메모"
permalink: /posts/
main_nav: true
---

{% for cat in site.descriptions %}
  <h3 id="{{cat}}">{{ cat }}</h3>
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
