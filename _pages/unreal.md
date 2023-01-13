---
title: Unreal
layout: collection
permalink: /unreal/
collection: unreal
entries_layout: grid
classes: wide
---

This is Unreal!

{% for unreal_article in site.unreal %}
<h2>
<a href="{{ unreal_article.url }}">
{{ unreal_article.title }}
{{ unreal_article.excerpt }}
</a>
</h2>
  <p>{{ unreal_article.content | markdownify }}</p>
{% endfor %}