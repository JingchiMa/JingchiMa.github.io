---
title: "Animes"
permalink: /animes/overview/
layout: single
---

{% for anime in site.animes %}
  <h2>{{ anime.name }} - {{ anime.rating }}</h2>
  <p>{{ anime.content | markdownify }}</p>
{% endfor %}
