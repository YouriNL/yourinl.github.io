---
layout: post
title: "Testing Jekyll Remote JSON Plugin"
categories: misc
---

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. 

Here is some JSON data: ( http://weerlive.nl/api/json-data-10min.php?key=demo&locatie=Amsterdam )
{{ site.data.weer_amsterdam.liveweer }}

<ul>
  {% for weer_amsterdam in site.data.weer_amsterdam.liveweer %}
    <li>Temp: {{ weer_amsterdam.temp }}</li>
    <li>Verwachting: {{ weer_amsterdam.verw }}</li>
  {% endfor %}    
</ul>

Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.