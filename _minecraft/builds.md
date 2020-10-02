---
layout: page
title: Server Builds
---
{% assign minecraft-build = site.minecraft | where: "type", "build" %}

{%- if minecraft-build.size > 0 -%}
{% for build in minecraft-build %}
<div class="minecraft"><h2><a href="{{ build.url }}">{{ build.title }}</a></h2></div>
{% endfor %}
{%- else -%}
<div class="minecraft"><h2>No build posts yet!</h2></div>
{%- endif -%}