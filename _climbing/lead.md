---
layout: page
title: Lead Climbing
---
{% assign climbing-gym = site.climbing | where: "type", "lead" %}

{%- if climbing-gym.size > 0 -%}
{% for climbing-gym in climbing-gym %}
<div class="climbing"><h2><a href="{{ climbing-gym.url }}">{{ climbing-gym.title }}</a></h2></div>
{% endfor %}
{%- else -%}
<div class="climbing"><h2>No build posts yet!</h2></div>
{%- endif -%}