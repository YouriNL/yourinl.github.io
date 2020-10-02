---
layout: page
title: Player Profiles
---
{% assign minecraft-player = site.minecraft | where: "type", "player" %}

{%- if minecraft-player.size > 0 -%}
{% for player in minecraft-player %}
<div class="minecraft"><h1><a href="{{ player.url }}">{{ player.title }}</a></h1></div>
{% endfor %}
{%- else -%}
<div class="minecraft"><h1>No player profiles yet!</h1></div>
{%- endif -%}