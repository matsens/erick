---
layout: default
title: Home
---

<div class="home">

  <ul class="posts">
    {% for post in site.posts %}
      <li>
        <a class="post-link" href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
        &nbsp;
        <span class="inline-post-date">{{ post.date | date: "%b %-d, %Y" }}</span>
      </li>
    {% endfor %}
  </ul>

</div>
