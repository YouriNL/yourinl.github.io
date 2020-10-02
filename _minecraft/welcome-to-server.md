---
layout: page
title: Welcome to the server.
type: minecraft-info
---
Hi, i'm a Vanilla server!

Here are some of our players:

{% assign minecraft-player = site.minecraft | where: "type", "player" %}
{%- if minecraft-player.size > 0 -%}
{% for player in minecraft-player %}
<div class="minecraft"><a href="{{ player.url }}">{{ player.title }}</a></div>
{% endfor %}
{%- else -%}
<div class="minecraft">No player profiles yet!</div>
{%- endif -%}