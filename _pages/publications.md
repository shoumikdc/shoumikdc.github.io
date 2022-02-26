---
layout: page
permalink: /Publications/
title: Publications
description: 
years: [2021, 2020, 2017]
nav: true
header-position: 3
---

<div class="publications">

{% for y in page.years %}
  <h2 class="year">{{y}}</h2>
  {% bibliography -f papers -q @*[year={{y}}]* %}
{% endfor %}

</div>
