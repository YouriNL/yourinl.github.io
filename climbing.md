---
layout: page
title: Climbing Gyms
---
{% for climbing in site.climbing %}
<div class="climbing"><h2><a href="{{ climbing.url }}">{{ climbing.title }}</a></h2></div>
{% endfor %}