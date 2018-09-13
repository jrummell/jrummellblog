---
layout: default
permalink: /tags
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

<h3 id="{{ t | downcase }}">{{ t | downcase }}</h3>
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
