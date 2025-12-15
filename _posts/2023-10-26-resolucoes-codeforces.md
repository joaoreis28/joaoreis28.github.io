---
title: "Índice de Resoluções de Problemas do Codeforces"
date: 2023-10-26 09:00:00 +0000
categories: [Codeforces]
tags: [índice, programação-competitiva]
pin: true
description: Índice de Resoluções de Problemas do Codeforces.
image : /assets/images/codeforces.jpg
---

Aqui você encontrará uma lista de todas as minhas resoluções de problemas do Codeforces. Esta página será atualizada a cada nova solução postada.

### Lista de Soluções

<ul>
  {% for post in site.categories.Codeforces %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a> - <span>{{ post.date | date: "%d/%m/%Y" }}</span>
    </li>
  {% endfor %}
</ul>