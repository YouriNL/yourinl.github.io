---
layout: default
---

<!-- Simple-Jekyll-Search HTML -->
<div id="search-container">
<input type="text" id="search-input" placeholder="Search blog posts...">
<ul id="results-container"></ul>
</div>

<!-- Simple-Jekyll-Search JS -->
<script src="/assets/js/simple-jekyll-search.min.js" type="text/javascript"></script>

<!-- Simple-Jekyll-Search Configuration -->
<script>
SimpleJekyllSearch({
  searchInput: document.getElementById('search-input'),
  resultsContainer: document.getElementById('results-container'),
  searchResultTemplate: '<div><a href="{url}"><h2>{title}</h2></a><span>{date}</span></div>',
  json: '/search.json',
  limit: 20,
  fuzzy: 1
})
noResultsText ("No result found!")
</script>