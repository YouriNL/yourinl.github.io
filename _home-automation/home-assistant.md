---
layout: page
title: Home Assistant
---
{% assign ha-ha = site.home-automation | where: "type", "home-assistant" %}

{%- if ha-ha.size > 0 -%}
{% for ha in ha-ha %}
<div class="home-automation"><h2><a href="{{ ha.url }}">{{ ha.title }}</a></h2></div>
{% endfor %}
{%- else -%}
<div class="home-automation"><h2>No Posts Yet!</h2></div>
{%- endif -%}