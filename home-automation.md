---
layout: page
title: Home Automation
---
{% for home-automation in site.home-automation %}
  <div class="home-automation">
    <h2><a href="{{ home-automation.url }}">{{ home-automation.title }}</a></h2>
  </div>
{% endfor %}