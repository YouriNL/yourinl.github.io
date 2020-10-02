---
layout: page
title: Node-Red
---
{% assign ha-nodered = site.home-automation | where: "type", "node-red" %}

{%- if ha-nodered.size > 0 -%}
{% for nodered in ha-nodered %}
<div class="home-automation"><h2><a href="{{ nodered.url }}">{{ nodered.title }}</a></h2></div>
{% endfor %}
{%- else -%}
<div class="home-automation"><h2>No Posts Yet!</h2></div>
{%- endif -%}