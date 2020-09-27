---
layout: page
title: Minecraft Server
---
All Posts:
{% for minecraft in site.minecraft limit:1 %}
<div class="minecraft"><h2><a href="{{ minecraft.url }}">{{ minecraft.title }}</a></h2></div>
{% endfor %}

{% assign minecraft-shop = site.minecraft | where: "type", "shop" %}

Shops:
{% for shop in minecraft-shop %}
<div class="minecraft"><h2><a href="{{ shop.url }}">{{ shop.title }}</a></h2></div>
{% endfor %}

{% assign minecraft-build = site.minecraft | where: "type", "build" %}

Builds:
{% for build in minecraft-build %}
<div class="minecraft"><h2><a href="{{ build.url }}">{{ build.title }}</a></h2></div>
{% endfor %}