---
layout: page
title: Climbing
---
All Posts:
{% for climbing in site.climbing %}
<div class="climbing"><h2><a href="{{ climbing.url }}">{{ climbing.title }}</a></h2></div>
{% endfor %}

{% assign boulder-gym = site.climbing | where: "type", "boulder" %}

Boulder Gyms:
{% for boulder-gym in boulder-gym %}
<div class="climbing"><h2><a href="{{ boulder-gym.url }}">{{ boulder-gym.title }}</a></h2></div>
{% endfor %}

{% assign climbing-gym = site.climbing | where: "type", "lead" %}

Climbing Gyms:
{% for climbing-gym in climbing-gym %}
<div class="climbing"><h2><a href="{{ climbing-gym.url }}">{{ climbing-gym.title }}</a></h2></div>
{% endfor %}