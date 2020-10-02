---
layout: page
title: ESP Home
---
{% assign ha-esphome = site.home-automation | where: "type", "esphome" %}

{%- if ha-esphome.size > 0 -%}
{% for esphome in ha-esphome %}
<div class="home-automation"><h2><a href="{{ esphome.url }}">{{ esphome.title }}</a></h2></div>
{% endfor %}
{%- else -%}
<div class="home-automation"><h2>No Posts Yet!</h2></div>
{%- endif -%}