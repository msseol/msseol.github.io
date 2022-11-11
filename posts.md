---
layout: page
title: "모든메모"
desc: "카테고리를 클릭하면 메모 목록이 나옵니다."
permalink: /posts/
main_nav: true
---

{% for cat in site.descriptions %}
  <h3 class="posts-list-title" id="{{cat}}" onclick="togglePosts('{{cat}}')">{{ cat }} <span class="post_count">{{site.categories[cat].size}}</span></h3>
  <ul class="posts-list" style="display:none" id="{{cat}}_post_wrap">
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

<script>
function togglePosts(id) {
  let postWrapId = id + "_post_wrap";
  let x = document.getElementById(postWrapId);
  if (x.style.display === "none") {
    x.style.display = "block";
  } else {
    x.style.display = "none";
  }
}
</script>