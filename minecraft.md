---
layout: page
title: Minecraft Server
---
{% assign minecraft-info = site.minecraft | where: "type", "minecraft-info" %}

{%- if minecraft-info.size > 0 -%}
{% for info in minecraft-info %}
<div class="minecraft">
    <h2>
        <a href="{{ info.url }}">{{ info.title }}</a>
    </h2>
</div>
{% endfor %}
{%- else -%}
<div class="minecraft"><h3>No information yet!</h3></div>
{%- endif -%}