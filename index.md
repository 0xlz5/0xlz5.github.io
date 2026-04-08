---
layout: page
title: Home
---

# 👋 Welcome

Hi, I’m Federico.  
This is my minimal GitHub Pages blog where I write notes, ideas, and experiments.

---

# 📝 Latest Posts

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
      <span style="color:#777;">— {{ post.date | date: "%Y-%m-%d" }}</span>
    </li>
  {% endfor %}
</ul>
