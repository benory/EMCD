---
layout: page
title: about
---
<head>
	<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"
	     integrity="sha256-p4NxAoJBhIIN+hmNHrzRCf9tD/miZyoHS5obTRR9BMY="
	     crossorigin=""/>
	<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"
     integrity="sha256-20nQCchB9co0qIjJZRGuk2/Z9VM+kNiyxNV1lvTlZBo="
     crossorigin=""></script>
</head>

<style>
	.grid {
		display: flex;
	}

	#map {
  		flex: 1;
  		height: 350px;
	}

	#map2 {
  		flex: 1;
  		height: 350px;
  		margin-left: 2em;
	}

</style>


We're often told how informal performance in European musicological seminars played important roles in the revival of medieval and Renaissance music performance following World War I. 

But this project argues that performance in more formal contexts was also essential in the rediscovery of early music. And a whole additional category of evidence for this survives in abundance, but has until now largely been ignored: programs for public concerts between 1915 and 1960—often introduced, directed, performed, or heard by the very same scholars leading important seminars—that included music from before 1600. To better understand the emergence of the early music performance tradition on both sides of the Atlantic, we need to systematically collect and curate all this information in a way that moves beyond narrative descriptions of a few individual concerts.

## The project

This project aims to meet this challenge by creating a relational database and mapping tool that will allow users to selectively display patterns of carefully-curated data from these programs. Users will be able to query the database to ask questions about the popularization of repertoires; the use of new performance and scholarly editions; the figures involved in the concerts; the early music objects that scholars and performers referenced and transcribed from; and the establishment of local, national, and international early music canons. 

Collecting this information is not simple: these programs are rarely published or digitized, but are instead often only found buried in personnel and institute files in university archives. This project seeks to collate and map materials, and provide context for important concerts. It also aims to enable users to flexibly manipulate this data to answer their own research questions.

### Why 1915 to 1960?

Sporadic performances of medieval and Renaissance music can be traced back to the ninteenth and even the eighteenth centuries, but until the 1910s, concerts of these repertoires were rare. All of this changed in Europe in the years following World War I: early music rapidly came into vogue. A generation of young scholars, captivated by informal music making movements such as the _Jugendmusikbewegung_ and emerging scholarship on music history, began to direct concerts and devote themselves to specialized knowledge about music from before 1600. The project's chronological bound of 1915 reflects this newfound interest.

After 1945, the center of early music scholarship shifted to the United States. Homegrown and émigré scholars alike led performance ensembles and taught generations of students who themselves came to focus on early music. Interest in early music on both sides of the Atlantic increased rapidly throughout the 1950s, and by the 1960s, the number performances had exploded. Ending the project's focus at 1960 enables the project to concentrate on the emergence of the modern early music tradition without being overwhelmed by the expansive concert data in ensuing decades.  

### Entering programs

For each program of early music:
+ works are identified by their standardized modern name, if possible; citations of modern editions are provided.
+ The probable composer for each work is listed, as agreed by modern scholars. Should there be more than one plausible conflicting attribution, these are provided as well.
+ music editions and original sources are named, if offered.
+ concert participants are identified.
+ details about performance location are entered.

### Documentation

For more details, read the [documentation](https://docs.google.com/document/d/18vVdL4CHMyDCxVk4t6r65NyTIwJbDcgxFDfYwpFgedg/edit){:target="_blank"} for the project (updated 19 May 2023).

### The archives

Concert programs have been located so far in the following archives:
<div class="grid">
	<div id="map">
		<script> 
			let archives = {% include_relative archives.json %};
			archives.archID      = "Archive ID (ARC)";
			archives.name        = "Name";		
			archives.archloc     = "Archive Location";
			archives.urlde       = "URL (DE)";
			archives.urlen       = "URL (EN)";

		//////////////////////////////
		//
		// domapsetup --
		//

		function doMapSetup() {

		let map = L.map('map').setView([50, 25], 4);
		map.options.minZoom = 4;

		L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
	    	maxZoom: 19,
	   		attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>'
		}).addTo(map);

		for (let i=0; i < archives.length; i++) {

			let url = "";
			if (archives[i][archives.urlde]) {
				url = archives[i][archives.urlde];
			} else if (archives[i][archives.urlen]) {
				url = archives[i][archives.urlen];
			}
			console.warn("url", url);
			console.warn("archive name", archives[i][archives.name]);
			console.warn("archive location", archives[i][archives.archloc]);

			if (archives[i][archives.archloc] && archives[i][archives.name]) {
				if (url) {
	     			L.marker([archives[i][archives.archloc]]).addTo(map).bindPopup(`<a target='_blank' href="${url}">${archives[i][archives.name]}</a>`);
	     		} else {
	     			L.marker([archives[i][archives.archloc]]).addTo(map).bindPopup(`${archives[i][archives.name]}`);
	     		}
     		}
		}

			/*
			L.marker([50.73420546539783, 7.102690461805324]).addTo(map)
	    		.bindPopup("<a target='_blank' href='https://www.uni-bonn.de/de/universitaet/organisation/weitere-einrichtungen/archiv-der-universitaet'>Universitätsarchiv Bonn</a>")
	    	L.marker([47.992586641284895, 7.845301862164243]).addTo(map)
	    		.bindPopup("<a target='_blank' href='https://www.uniarchiv.uni-freiburg.de'>Universitätsarchiv Freiburg</a>")
	    	L.marker([52.438027825357835, 13.53445588334833]).addTo(map)
	    		.bindPopup("<a target='_blank' href='https://www.ub.hu-berlin.de/de/standorte/archiv'>Universitätsarchiv Humboldt Universität</a>")
	    	L.marker([51.33587250631357, 12.388703512116019]).addTo(map)
	    		.bindPopup("<a target='_blank' href='https://www.universitaetsarchivleipzig.de'>Universitätsarchiv Leipzig</a>")
	    	L.marker([48.16063263366055, 11.562894375163575]).addTo(map)
	    		.bindPopup("<a target='_blank' href='https://stadt.muenchen.de/rathaus/verwaltung/direktorium/stadtarchiv.html'>Stadtarchiv München</a>")
	    	L.marker([56.505583949298945, 13.047239412811416]).addTo(map)
	    		.bindPopup('Privat Nachlass Walter Gerstenberg')
	    	*/
		}

		document.addEventListener("DOMContentLoaded",  function () {
     			doMapSetup();
			});


		</script>
	</div>
	<div id="map2">
		<script> 
			let map2 = L.map('map2').setView([37, -100], 3);
			map2.options.minZoom = 3;

			L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
	    		maxZoom: 19,
	   			attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>'
			}).addTo(map2);
			L.marker([42.373116227949886, -71.11528917862246]).addTo(map2)
	    		.bindPopup("<a target='_blank' href='https://library.harvard.edu/collections/harvard-theatre-collection'>Harvard Theatre Collection, Houghton Library</a>")
	    	L.marker([40.773567244715764, -73.98410683081381]).addTo(map2)
	    		.bindPopup("<a target='_blank' href='https://www.nypl.org/locations/lpa/music-division'>New York Public Library for the Performing Arts, Music Division</a>")
	    	L.marker([41.79541254876555, -87.59226344635543]).addTo(map2)
	    		.bindPopup("<a target='_blank' href='https://www.lib.uchicago.edu/scrc/'>University of Chicago Special Collections</a>")
	    	L.marker([37.870596548069116, -122.25578955689716]).addTo(map2)
	    		.bindPopup("<a target='_blank' href='https://www.lib.berkeley.edu/visit/music'>University of California, Berkeley, Music Library</a>")
		</script>
	</div>	
</div>

<br>

Do you know of concert programs in archives that should be included in this project? [Let us know](mailto:concertsdatabase@gmail.com).

### Support

This project has been supported by a 2023 [Ora Frishberg Saloman Fund Award from the American Musicological Society](https://www.amsmusicology.org/page/Saloman_Winners){:target="_blank"}.