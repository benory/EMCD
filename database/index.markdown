---
layout: page
title: concerts database
---

{% include_relative scripts-local.html %}
{% include_relative styles-local.html %}

<div id="sheet-select">
	<span class="sheet-button selected" data-sheet="works">Works</span>
	<span class="sheet-button" data-sheet="concerts">Concerts</span>
</div>

<input type="text" id="input" onkeyup="UserSearch()" placeholder="Enter title, composer, source, or date"><span id="search-count"></span>

<div id="browse-interface">
	<div class="sheet-display" data-sheet="works">
		<div class="search-interface"></div>
		<div class="results-list"></div>
	</div>
	<div class="sheet-display hidden" data-sheet="concerts">
		<div class="search-interface"></div>
		<div class="results-list"></div>
	</div>
</div>



