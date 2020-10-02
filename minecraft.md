---
layout: page
title: Minecraft Server
---
{% for minecraft in site.minecraft %}
<div class="minecraft">
    <h2>
        <a href="{{ minecraft.url }}">{{ minecraft.title }}</a>
    </h2>
</div>
{% endfor %}