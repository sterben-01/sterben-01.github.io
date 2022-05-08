---
layout: tags
title: 标签
icon: fas fa-tags
order: 2
---

```HTML
<div>
  <h1>Articles tagged with ""</h1>
  <ul style='padding-top: 16px;'>
  
  {% for post in site.posts %}
    {% if post.tags contains page.tag-name %}
      <li><a href="{{ post.url }}">{{ post.title }}</a>, published {{ post.date | date: "%Y-%m-%d" }}</li>
    {% endif %}
  {% endfor %}
  
  </ul>
</div>
```

