---
layout: page
title: Shop Information
---
{% assign minecraft-shop = site.minecraft | where: "type", "shop" %}

{%- if minecraft-shop.size > 0 -%}
{% for shop in minecraft-shop %}
<div class="minecraft"><h2><a href="{{ shop.url }}">{{ shop.title }}</a></h2></div>
{% endfor %}
{%- else -%}
<div class="minecraft"><h2>No shop information yet!</h2></div>
{%- endif -%}