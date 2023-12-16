---
layout: page
title: 카테고리
description: 📁모든 카테고리의 링크를 확인할 수 있습니다
background: '/img/bg-category-2.jpg'
---

<header class="site-category">
  <ul>
    
    {% assign pages_list = site.pages %}
    {% for node in pages_list %}
      {% if node.title != null %}
        {% if node.layout == "category" %}
          <li><a class="category-link {% if page.url == node.url %} active{% endif %}"
          href="{{ site.baseurl }}{{ node.url }}">{{ node.title }}</a></li>
        {% endif %}
      {% endif %}
    {% endfor %}
    
</ul>
</header>