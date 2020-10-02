---
layout: page
title: Tasmota
---
{% assign ha-tasmota = site.home-automation | where: "type", "tasmota" %}

{%- if ha-tasmota.size > 0 -%}
{% for tasmota in ha-tasmota %}
<div class="home-automation"><h2><a href="{{ tasmota.url }}">{{ tasmota.title }}</a></h2></div>
{% endfor %}
{%- else -%}
<div class="home-automation"><h2>No Posts Yet!</h2></div>
{%- endif -%}