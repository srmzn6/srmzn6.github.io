---
layout: page
title: Blog
---
<p>
	<h3> Some of my thoughts and experiences over the years are listed as blog posts.</h3>
</p>

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
	  <p class="date">{{ post.date | date: "%b %d, %Y" }}</p>
    </li>
  {% endfor %}
</ul>