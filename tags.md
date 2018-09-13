---
layout: default
permalink: /xval-webforms
redirect_from:
 - /tag
 - /tag/
 - /blog/index.php/tag/xval-webforms
 - /blog/index.php/tag/xval-webforms/
 - /blog/tag/xval-webforms
 - /blog/tag/xval-webforms/
 - /tag/xval-webforms/
 - /tag/xval-webforms
 - /tag/facebook-steam-achievements/
 - /tag/facebook-steam-achievements
 - /tag/jquery-message/
 - /tag/jquery-message
title: Tags
---

{% for tag in site.tags %}
  {% assign t = tag | first %}
  {% assign posts = tag | last %}

{{ t | downcase }}
<ul>
{% for post in posts %}
  {% if post.tags contains t %}
  <li>
    <a href="{{ post.url }}">{{ post.title }}</a>
    <span class="date">{{ post.date | date: "%B %-d, %Y"  }}</span>
  </li>
  {% endif %}
{% endfor %}
</ul>
{% endfor %}
