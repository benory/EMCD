---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: page 
---

<script src="https://cdn.jsdelivr.net/npm/vega@5.25.0"></script>
<script src="https://cdn.jsdelivr.net/npm/vega-lite@5.15.1"></script>
<script src="https://cdn.jsdelivr.net/npm/vega-embed@6.22.2"></script>

{% include_relative styles-local.html %}
{% include_relative scripts-local.html %}

The Early Music Concerts Database catalogues concert programs of medieval and Renaissance music from roughly 1915 through 1960. It aims to answer questions of how the early music canon was formed and how scholars, performers, and audiences alike rediscovered long-lost repertoires. Read [about this project](about) or [browse the database](database).<br><br>

#### Concert Locations

<!-- <div id="mapSelect">
   <div onclick="display()" class="button hidden">Show US Concerts</div>
   <div onclick="display()" class="button">Show European Concerts</div>
</div><br> -->

<div class="grid">
	<div id="map"></div>
	<div id="map2"></div>
</div> <br>

#### Project data
+ <span id="concert-count"></span> catalogued concerts
+ <span id="work-count"></span> individual performed works, including pieces by <span id="composer-count"></span> individual composers 
+ <span id="person-count"></span> musicologists, singers, and musicians catalogued

<div id="concerts-by-year"></div>