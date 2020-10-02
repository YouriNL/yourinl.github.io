---
layout: page
title: Player Profiles
---
{% assign minecraft-player = site.minecraft | where: "type", "player" %}

{%- if minecraft-player.size > 0 -%}
{% for player in minecraft-player %}
<div class="minecraft"><h2><a href="{{ player.url }}">{{ player.title }}</a></h2></div>
{% endfor %}
{%- else -%}
<div class="minecraft"><h2>No player profiles yet!</h2></div>
{%- endif -%}