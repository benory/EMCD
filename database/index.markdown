---
layout: page
title: database
---

<style>
	body {font: 400 12px/1 -apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,Helvetica,Arial,sans-serif,"Apple Color Emoji","Segoe UI Emoji","Segoe UI Symbol"}
	h1 { font-size: 40px; }
	th { text-align: left; }
	table.browse-works, table.browse-concerts { min-width: 1000px;}
	table.browse-works, table.browse-concerts { margin-left: auto; margin-right: auto; } /* center table */
	table.browse-works, table.browse-concerts { border-collapse: collapse; } /* don't put gaps between cells */
	table.browse-works th, table.browse-concerts th { background:skyblue; }
	table.browse-works td, table.browse-concerts td, table.browse th, table.browse-concerts th {padding-left: 2px; padding-top: 2px; padding: 2px}
	table.browse-works tr:hover, table.browse-concerts tr:hover { background:#ff000011; }
	a { text-decoration: none; }
	div.search-interface { margin-top: 30px; margin-bottom: 30px; }
	.wrapper {margin-left: 10px;}
	table.browse-works td:nth-child(2) {min-width: 125px;}
	table.browse-works td:nth-child(4) {white-space: nowrap;}
	table.browse-works td:nth-child(5) {min-width: 100px}
	table.browse-works td:nth-child(6) {min-width: 150px;}
	table.browse-works td:nth-child(7) {min-width: 200px;}
	table.browse-concerts td:nth-child(1) {white-space: nowrap;}
	table.browse-concerts td:nth-child(2) {min-width: 250px;}
	table.browse-concerts td:nth-child(3) {min-width: 200px;}
	table.browse-concerts td:nth-child(4) {min-width: 200px;}
	table.browse-concerts td:nth-child(5) {min-width: 200px;}
	select.source {max-width: 250px}
	span.sheet-button {
		font: 400 18px/1 -apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,Helvetica,Arial,sans-serif,"Apple Color Emoji","Segoe UI Emoji","Segoe UI Symbol";
		color: #0645AD;
		display: inline-block;
		padding-bottom: 25px;
		padding-right: 25px;
	}
	span.sheet-button:hover {
 		text-decoration: underline;
 	}

</style>

<script>

//////////////////////////////
//
// Click manager for selecting which worksheet data to browse:
//

document.addEventListener("click", function (event) {
	let clickedElement = event.target;
	let targetButton = clickedElement.closest(".sheet-button");
	if (!targetButton) {
		return;
	}
	let name = targetButton.dataset.sheet;
	displaySheet(name);
});



//////////////////////////////
//
// displaySheet -- Select the browse interface for a specific worksheet.
//

function displaySheet(name) {
	let list = document.querySelectorAll(".sheet-display");
	for (let i=0; i<list.length; i++) {
		let sheet = list[i];
		let sheetName = sheet.dataset.sheet;
		sheet.style.display = (name == sheetName ? "block" : "none");
	}
}


</script>


<div id="sheet-select">
	<span class="sheet-button" data-sheet="works">Works</span>
	<span class="sheet-button" data-sheet="concerts">Concerts</span>
</div>

<div id="browse-interface">
	<div class="sheet-display" data-sheet="works">
		<div class="search-interface"></div>
		<div class="results-list"></div>
	</div>
	<div class="sheet-display" data-sheet="concerts">
		<div class="search-interface"></div>
		<div class="results-list"></div>
	</div>
</div>

<script>
// vim: ts=3:nowrap

let EMC = {};
EMC.results = {};  // elements for displaying search results by sheet name.
EMC.menus = {}; // elements for displaying search menus by sheet name.
EMC.activeResults = null;
EMC.index = {};    // header name mapping by sheet.
EMC.index.works = {};  // header names for works sheet.
EMC.index.concerts = {};  // header names for works concerts.
EMC.index.archives = {}; // header names for archives sheet.
EMC.lookup = {};
EMC.METADATA = {};
EMC.METADATA.works = {% include_relative works.json %};
EMC.METADATA.concerts = {% include_relative concerts.json %};
EMC.METADATA.archives = {% include_relative archives.json %};

EMC.index.works.name          = "Standardized Name of Work";
EMC.index.works.composer      = "Probable Composer";
EMC.index.works.voices        = "Voices";
EMC.index.works.composername  = "Composer Name as Listed in Program";
EMC.index.works.conflattr     = "Conflicting Attributions";
EMC.index.works.language	   = "Language";
EMC.index.works.language2	   = "Second Language";
EMC.index.works.monopoly	   = "Monophonic/Polyphonic";
EMC.index.works.sacrsec	      = "Sacred/Secular";
EMC.index.works.vocinstr	   = "Vocal/Instrumental";
EMC.index.works.genre   	   = "Genre";
EMC.index.works.source    	   = "Source of Work Listed in Program";
EMC.index.works.folios	      = "Folios/No.";
EMC.index.works.edition	      = "Edition of Work Listed in Program";
EMC.index.works.pages	      = "Nos./Page Numbers";
EMC.index.works.scanedition   = "Scan of Edition";
EMC.index.works.ProgID	      = "Program ID";
EMC.index.works.ProgDate	   = "Program Date";
EMC.index.works.ProgOrder	   = "Order in Program";
EMC.index.works.NotesWork	   = "Notes on Work";
EMC.index.works.ModernEd	   = "Modern Edition";
EMC.index.works.Repeatcon	   = "Repeat Concerts";
EMC.index.concerts.ID          = "ID";
EMC.index.concerts.year        = "Year";
EMC.index.concerts.month       = "Month";
EMC.index.concerts.day         = "Day";
EMC.index.concerts.date        = "Date";
EMC.index.concerts.ProgTitle   = "Program Title";
EMC.index.concerts.ensemble    = "Ensemble/Larger Org.";
EMC.index.concerts.loc         = "Location";
EMC.index.concerts.city        = "City";
EMC.index.concerts.state       = "State";
EMC.index.concerts.country     = "Country";
EMC.index.concerts.gmaps       = "Google Maps URL";
EMC.index.concerts.loccoord    = "Location Coordinates";
EMC.index.concerts.intro       = "Introduction";
EMC.index.concerts.direction   = "Direction";
EMC.index.concerts.performers  = "Performers";
EMC.index.concerts.archive     = "Archive (ARC)";
EMC.index.concerts.signature   = "Signature";
EMC.index.concerts.notes       = "Notes on Program";
EMC.index.concerts.literature  = "Literature";
EMC.index.concerts.image       = "Image";
EMC.index.concerts.extimage    = "Externally Hosted Image";
EMC.index.archives.archID      = "Archive ID (ARC)";
EMC.index.archives.country     = "Country";
EMC.index.archives.name        = "Name";
EMC.index.archives.urlde       = "URL (DE)";
EMC.index.archives.urlen       = "URL (EN)";
EMC.index.archives.archloc     = "Archive Location";

document.addEventListener("DOMContentLoaded", function () {
	buildLookupTables();
	buildSearchInterfaces(EMC.METADATA, "#browse-interface");
	displayBrowseTableWorks(EMC.METADATA.works);
	displayBrowseTableConcerts(EMC.METADATA.concerts);
});

//////////////////////////////
//
// buildLookupTables –- 
//

function buildLookupTables() {
	let metadata = EMC.METADATA;
	if (!metadata){
		console.warn("No METADATA!");
		return;
	}
	for (sheet in metadata) {
		if (sheet === "works"){
			continue;
		}
		buildLookupTable(sheet);
	}
}

//////////////////////////////
//
// buildLookupTable –-
//

function buildLookupTable(sheet) {
	let sheetArray = EMC.METADATA[sheet];
	if (!sheetArray && Array.isArray(sheetArray)){
		console.warn("No METADATA FOR", sheet);
		return;
	}
	EMC.lookup[sheet] = {};
	const lookup = EMC.lookup[sheet];
	for (let entry of sheetArray) {
		let id = entry.ID;
		if (!id){
			console.warn("NO ID FOR ENTRY");
			continue;
		}
		lookup[id] = entry;
	}
}

	

//////////////////////////////
//
// buildSearchInterfaces --
//

function buildSearchInterfaces(metadata, selector) {
	let element = document.querySelector(selector);
	if (!element) {
		console.error("ERROR: Cannot find", selector, "element");
		return;
	}
	let browsers = element.querySelectorAll("div.sheet-display");
	for (let i=0; i<browsers.length; i++) {
		let sheetName = browsers[i].dataset.sheet;
		let browseElement = browsers[i].querySelector("div.search-interface");
		if (!browseElement) {
			console.error("ERROR: No browseElement for", sheetName);
			return;
		}
		EMC.menus[sheetName] = browseElement;
		let tableElement = browsers[i].querySelector("div.results-list");
		if (!tableElement) {
			console.error("ERROR: No search results list element for", sheetName);
			return;
		}
		EMC.results[sheetName] = tableElement;
		if (sheetName === "works") {
			buildSearchInterfaceWorks(metadata.works, browseElement);
		} else if (sheetName === "concerts") {
			buildSearchInterfaceConcerts(metadata.concerts, browseElement);
		}
	}
}


//////////////////////////////
//
// buildSearchInterfaceWorks --
//

function buildSearchInterfaceWorks(data, browseElement) {
	if (!browseElement) {
		console.error("ERROR: Cannot find search interface element", browseElement);
		return;
	}
	let output = "";
	output += buildComposerSelect(data);
	output += buildVoiceSelect(data);
	output += buildGenreSelect(data);
	output += buildLanguageSelect(data);
	output += buildMonoPolySelect(data);
	output += buildSacredSecularSelect(data);
	output += buildVocInstrSelect(data);
	output += buildSourceSelect(data);
	browseElement.innerHTML = output;
}


//////////////////////////////
//
// buildSearchInterfaceConcerts --
//

function buildSearchInterfaceConcerts(data, browseElement) {
	if (!browseElement) {
		console.error("ERROR: Cannot find search interface element", browseElement);
		return;
	}
	let output = "";
	output += buildCountrySelect(data);
	output += buildYearSelect(data);
	output += buildProgramSourceSelect(data);
	browseElement.innerHTML = output;
}


//////////////////////////////
//
// displayBrowseTableWorks --
//

function displayBrowseTableWorks(data) {
	let element = EMC.results.works;
	if (!element) {
		console.warn("Cannot find search results element for works");
		return;
	}

	let headings = [EMC.index.works.name, EMC.index.works.composer,
	EMC.index.works.voices, EMC.index.works.ProgDate,
	EMC.index.works.genre, EMC.index.works.source,
	EMC.index.works.edition, EMC.index.works.ModernEd];

	let contents = "";
	contents += "<table class='browse-works'>\n";
	contents += "<thead>\n";
	contents += makeTableHeader(headings);
	contents += "</thead>\n";
	contents += "<tbody>\n";
	contents += makeTableBody(headings, data);
	contents += "</tbody>\n";
	contents += "</table>\n";
	element.innerHTML = contents;
}

//////////////////////////////
//
// displayBrowseTableConcerts --
//

function displayBrowseTableConcerts(data) {
	let element = EMC.results.concerts;
	if (!element) {
		console.warn("Cannot find search results element for works");
		return;
	}

	let headings = [EMC.index.concerts.date, EMC.index.concerts.ProgTitle, EMC.index.concerts.ensemble, EMC.index.concerts.loc, EMC.index.concerts.direction, EMC.index.concerts.archive, EMC.index.concerts.signature];

	let contents = "";
	contents += "<table class='browse-concerts'>\n";
	contents += "<thead>\n";
	contents += makeTableHeader(headings);
	contents += "</thead>\n";
	contents += "<tbody>\n";
	contents += makeTableBody(headings, data);
	contents += "</tbody>\n";
	contents += "</table>\n";
	element.innerHTML = contents;
}

//////////////////////////////
//
// makeTableHeader -- Generate HTML content for browse table header.
//

function makeTableHeader(headings) {
	let output = "";
	let archivename = "Program Source";
	for (let i=0; i<headings.length; i++ ) {
		output += "<th>";
		if (headings[i] == EMC.index.concerts.archive){
			output += archivename;
		} else {
			output += headings[i];
		}
		output += "</th>";
	}
	return output;
}



//////////////////////////////
//
// makeTableBody -- Generate HTML content for browse table's body.
//

function makeTableBody(headings, data) {
	let output = "";
	for (let i=0; i<data.length; i++) {
		let entry = data[i];
		output += "<tr>";
		for (let i=0; i<headings.length; i++) {
			let value = "";
			if (typeof entry[headings[i]] !== "undefined") {
				value = entry[headings[i]];
			}
			output += "<td>";

			if (headings[i] == EMC.index.works.edition) {
				let editioncombined = getEdition(entry);
				let url = getEditionUrl(entry);
				output += `<a target="_blank" href="${url}">${editioncombined}</a>`;
			} else if (headings[i] == EMC.index.works.source) {
				let sourcecombined = getSource(entry);
				output += sourcecombined;
			} else if (headings[i] == EMC.index.concerts.ProgTitle) {
				let ProgTitle = value;
				let ProgUrl = getProgUrl(entry);
				output += `${ProgTitle} [<a target="_blank" href="${ProgUrl}">Image</a>]`;
			} else if (headings[i] == EMC.index.concerts.loc) {
				let loccombined = getLocation(entry);
				let locmaps = getLocationGoogleMaps(entry);
 				output += `<a target="_blank" href="${locmaps}">${loccombined}</a>`;
			} else if (headings[i] == EMC.index.concerts.direction){
				let directioncleaned = getCleanedDirection(entry);
				output += directioncleaned;
			} else if (headings[i] == EMC.index.concerts.archive) {
				if (value){
					if (value.match(";")){
						value = value.trim().split(/\s*;\s*/);
						console.warn("value match", value);
					} else {
						value = [ value ];
						console.warn("value", value);
					}
					console.warn("value", value);
					for (let i=0; i<value.length; i++){
						let aentry = EMC.lookup.archives[value[i]];
						console.warn("aentry", aentry);
						let name = aentry[EMC.index.archives.name];
						console.warn("name", name);
						output += name;
						if (i < value.length - 1){
							output += "; ";
						}
					}
				}
			} else {
				output += value;
			}
			output += "</td>";
		}
		output += "</tr>\n";
	}
	return output;
}



//////////////////////////////
//
// buildComposerSelect --
//

function buildComposerSelect(data) {
	let counter = {};
	let sum = data.length;
	for (let i=0; i<sum; i++) {
		let entry = data[i];
		let composer = entry[EMC.index.works.composer];
		if (!composer) {
			//console.error("WARNING: ", entry, " DOES NOT HAVE A COMPOSER");
			continue;
		}
		counter[composer] = (counter[composer] === undefined) ? 1 : counter[composer] + 1;
	}

	let clist = Object.keys(counter).sort();
	clist.sort((a, b) => a.toLowerCase().localeCompare(b.toLowerCase()));
	let composerCount = clist.length;
	let output = "<select class='composer' onchange='doSearchWorks()'>\n";
	output += `<option value="">Any composers [${composerCount}]</option>`;
	for (let i=0; i<clist.length; i++) {
		let name = clist[i];
		let count = counter[clist[i]];
		output += `<option value="${name}">${name} (${count})</option>`;
	}
	output += "</select>\n";
	return output;
}


//////////////////////////////
//
// buildGenreSelect -- generate menu for genres, sort by count
//

function buildGenreSelect(data) {
	let genres = {};
	for (let entry of EMC.METADATA.works) {
		let genre = entry.Genre;
		if (typeof genres[genre] !== "undefined") {
			genres[genre]++;
		} else {
			genres[genre] = 1;
		}
	}

	let keys = Object.getOwnPropertyNames(genres);
	keys.sort((a, b) => {
		if (genres[a] == genres[b]) {
			// sort cases alphabetically by genre if the have the same count:
			return a.localeCompare(b);
		} else {
			return genres[b] - genres[a];
		}
	});
	let genreCount = keys.length;

	let output = "<select class='genre' onchange='doSearchWorks()'>\n";
	output += `<option value=''>Any genre [${genreCount}]</options>`;
	for (let genre of keys) {
		if (genre !== "undefined") {
			output += `<option value="${genre}">${genre} (${genres[genre]})</option>`;
		}
	}
	output += "</select>";
	return output;
}

//////////////////////////////
//
// getSource -- Generate Source + Folios/Pages 
//

function getSource(entry) {
	let source = "";
	if (typeof entry["Source of Work Listed in Program"] !== "undefined") {
		source = entry["Source of Work Listed in Program"];
	}
	let sourcepages = "";
	if (typeof entry["Folios/No."] !== "undefined") {
		sourcepages = entry["Folios/No."];
	}
	if (!sourcepages.match(/^\s*$/)) {
		if (!source.match(/^\s*$/)) {
			return `${source}, ${sourcepages}`;
		} else {
			return `${sourcepages}`;
		}
	}
	if (source.match(/^\s*$/)) {
		return "";
	} else {
		return source;
	}
}

//////////////////////////////
//
// getEdition -- Generate Edition + Pages 
//

function getEdition(entry) {
	let edition = "";
	if (typeof entry["Edition of Work Listed in Program"] !== "undefined") {
		edition = entry["Edition of Work Listed in Program"];
	}
	let editionpages = "";
	if (typeof entry["Nos./Page Numbers"] !== "undefined") {
		editionpages = entry["Nos./Page Numbers"];
	}
	if (!editionpages.match(/^\s*$/)) {
		if (!edition.match(/^\s*$/)) {
			return `${edition}, ${editionpages}`;
		} else {
			return `${editionpages}`;
		}
	}
	if (edition.match(/^\s*$/)) {
		return "";
	} else {
		return edition;
	}
}

//////////////////////////////
//
// getEditionUrl -- Generate a source link based on "Scan of Edition".
//

function getEditionUrl(entry) {
	let editionurl = "";
	if (typeof entry["Scan of Edition"] !== "undefined") {
		editionurl = entry["Scan of Edition"];
		return editionurl;
	}
	return "";
}

//////////////////////////////
//
// buildLanguageSelect --
//

function buildLanguageSelect(data) {
	let counter = {};
	let sum = data.length;
	for (let i=0; i<sum; i++) {
		let entry = data[i];
		let language = entry[EMC.index.works.language];
		if (!language) {
			//console.error("WARNING: ", entry, " DOES NOT HAVE A LANGUAGE");
			continue;
		}
		counter[language] = (counter[language] === undefined) ? 1 : counter[language] + 1;
	}

	let llist = Object.keys(counter).sort();
	let languageCount = llist.length;
	let output = "<select class='language' onchange='doSearchWorks()'>\n";
	output += `<option value="">Any language [${languageCount}]</option>`;
	for (let i=0; i<llist.length; i++) {
		let name = llist[i];
		let count = counter[llist[i]];
		output += `<option value="${name}">${name} (${count})</option>`;
	}
	output += "</select>\n";
	return output;
}


//////////////////////////////
//
// buildMonoPolySelect --
//

function buildMonoPolySelect(data) {
	let counter = {};
	let sum = data.length;
	for (let i=0; i<sum; i++) {
		let entry = data[i];
		let monopoly = entry[EMC.index.works.monopoly];
		if (!monopoly) {
			//console.error("WARNING: ", entry, " DOES NOT HAVE A MONOPHONIC/POLYPHONIC DESIGNATION");
			continue;
		}
		counter[monopoly] = (counter[monopoly] === undefined) ? 1 : counter[monopoly] + 1;
	}

	let mlist = Object.keys(counter).sort();
	let monopolyCount = mlist.length;
	let output = "<select class='monopoly' onchange='doSearchWorks()'>\n";
	output += `<option value="">monophonic/polyphonic [${monopolyCount}]</option>`;
	for (let i=0; i<mlist.length; i++) {
		let name = mlist[i];
		let count = counter[mlist[i]];
		output += `<option value="${name}">${name} (${count})</option>`;
	}
	output += "</select>\n";
	return output;
}


//////////////////////////////
//
// buildSacredSecularSelect --
//

function buildSacredSecularSelect(data) {
	let counter = {};
	let sum = data.length;
	for (let i=0; i<sum; i++) {
		let entry = data[i];
		let sacredsecular = entry[EMC.index.works.sacrsec];
		if (!sacredsecular) {
			//console.error("WARNING: ", entry, " DOES NOT HAVE A SACRED/SECULAR DESIGNATION");
			continue;
		}
		counter[sacredsecular] = (counter[sacredsecular] === undefined) ? 1 : counter[sacredsecular] + 1;
	}

	let slist = Object.keys(counter).sort();
	let sacredsecularCount = slist.length;
	let output = "<select class='sacredsecular' onchange='doSearchWorks()'>\n";
	output += `<option value="">sacred/secular [${sacredsecularCount}]</option>`;
	for (let i=0; i<slist.length; i++) {
		let name = slist[i];
		let count = counter[slist[i]];
		output += `<option value="${name}">${name} (${count})</option>`;
	}
	output += "</select>\n";
	return output;
}


//////////////////////////////
//
// buildVocInstrSelect --
//

function buildVocInstrSelect(data) {
	let counter = {};
	let sum = data.length;
	for (let i=0; i<sum; i++) {
		let entry = data[i];
		let vocinstr = entry[EMC.index.works.vocinstr];
		if (!vocinstr) {
			//console.error("WARNING: ", entry, " DOES NOT HAVE A VOCAL/INSTRUMENTAL DESIGNATION");
			continue;
		}
		counter[vocinstr] = (counter[vocinstr] === undefined) ? 1 : counter[vocinstr] + 1;
	}

	let vilist = Object.keys(counter).sort();
	let vocinstrCount = vilist.length;
	let output = "<select class='vocinstr' onchange='doSearchWorks()'>\n";
	output += `<option value="">vocal/instrumental [${vocinstrCount}]</option>`;
	for (let i=0; i<vilist.length; i++) {
		let name = vilist[i];
		let count = counter[vilist[i]];
		output += `<option value="${name}">${name} (${count})</option>`;
	}
	output += "</select>\n";
	return output;
}

//////////////////////////////
//
// buildSourceSelect --
//

function buildSourceSelect(data) {
	let counter = {};
	let sum = data.length;
	for (let i=0; i<sum; i++) {
		let entry = data[i];
		let source = entry[EMC.index.works.source];
		if (!source) {
			//console.error("WARNING: ", entry, " DOES NOT HAVE A SOURCE");
			continue;
		}
		counter[source] = (counter[source] === undefined) ? 1 : counter[source] + 1;
	}

	let solist = Object.keys(counter).sort();
	let sourceCount = solist.length;
	let output = "<select class='source' onchange='doSearchWorks()'>\n";
	output += `<option value="">Any source [${sourceCount}]</option>`;
	for (let i=0; i<solist.length; i++) {
		let name = solist[i];
		let count = counter[solist[i]];
		output += `<option value="${name}">${name} (${count})</option>`;
	}
	output += "</select>\n";
	return output;
}

//////////////////////////////
//
// buildVoiceSelect --
//

function buildVoiceSelect(data) {
	let counter = {};
	let fileCount = data.length;
	for (let i=0; i<fileCount; i++) {
		let entry = data[i];
		let voice = entry[EMC.index.works.voices];
		if (!voice) {
			//console.error("WARNING: ", entry, " DOES NOT HAVE A VOICE COUNT");
			continue;
		}
		counter[voice] = (counter[voice] === undefined) ? 1 : counter[voice] + 1;
	}

	let vlist = Object.keys(counter).sort();
	vlist.sort((a, b) => (a - b));
	let output = "<select class='voice' onchange='doSearchWorks()'>\n";
	output += `<option value="">Any voice count</option>`;
	for (let i=0; i<vlist.length; i++) {
		let vcount = vlist[i];
		output += `<option value="${vcount}">${vcount}</option>`;
	}
	output += "</select>\n";
	return output;
}

//////////////////////////////
//
// getProgUrl -- Generate a source link based on "Scan of Edition".
//

function getProgUrl(entry) {
	let ProgUrl = "";
	if (typeof entry["Image"] !== "undefined") {
		ProgUrl = entry["Image"];
		return ProgUrl;
	}
	return "";
	console.warn("ProgUrl", ProgUrl);
}

//////////////////////////////
//
// getLocation -- Generate Location + City + Country
//

function getLocation(entry) {
	let location = "";
	let city = "";
	let country = "";
	if (typeof entry["Location"] !== "undefined") {
		location = entry["Location"];
	}
	if (typeof entry["City"] !== "undefined") {
		city = entry["City"];
	}
	if (typeof entry["Country"] !== "undefined") {
		country = entry["Country"];
	}
	if (!location.match(/^\s*$/) && !city.match(/^\s*$/) && !country.match (/^\s*$/)) {
		return `${location}, ${city}, ${country}`;
	} else if (!location.match(/^\s*$/) && !country.match (/^\s*$/)){
		return `${location}, ${country}`;
	}
	if (location.match(/^\s*$/)) {
		return "";
	} else {
		return location;
	}
}

//////////////////////////////
//
// getLocationGoogleMaps -- Generate a source link based on "Scan of Edition".
//

function getLocationGoogleMaps(entry) {
	let locmapsurl = "";
	if (typeof entry["Google Maps URL"] !== "undefined") {
		locmapsurl = entry["Google Maps URL"];
		return locmapsurl;
	}
	return "";
}

//////////////////////////////
//
// getCleanedDirection -- Remove {}.
//

function getCleanedDirection(entry) {
	let cleandirection = "";
	if (typeof entry["Direction"] !== "undefined") {
		cleandirection = entry["Direction"].replace(/{/g, '');
		cleandirection = cleandirection.replace(/}/g, '');
		return cleandirection;
	}
	return "";
}

//////////////////////////////
//
// buildCountrySelect --
//

function buildCountrySelect(data) {
	let counter = {};
	let sum = data.length;
	for (let i=0; i<sum; i++) {
		let entry = data[i];
		let country = entry[EMC.index.concerts.country];
		if (!country) {
			//console.error("WARNING: ", entry, " DOES NOT HAVE A COUNTRY DESIGNATION");
			continue;
		}
		counter[country] = (counter[country] === undefined) ? 1 : counter[country] + 1;
	}

	let clist = Object.keys(counter).sort();
	let country = clist.length;
	let output = "<select class='country' onchange='doSearchConcerts()'>\n";
	output += `<option value="">Country [${country}]</option>`;
	for (let i=0; i<clist.length; i++) {
		let name = clist[i];
		let count = counter[clist[i]];
		output += `<option value="${name}">${name} (${count})</option>`;
	}
	output += "</select>\n";
	return output;
}


//////////////////////////////
//
// buildYearSelect --
//

function buildYearSelect(data) {
	let counter = {};
	let sum = data.length;
	for (let i=0; i<sum; i++) {
		let entry = data[i];
		let year = entry[EMC.index.concerts.year];
		if (!year) {
			//console.error("WARNING: ", entry, " DOES NOT HAVE A YEAR DESIGNATION");
			continue;
		}
		counter[year] = (counter[year] === undefined) ? 1 : counter[year] + 1;
	}

	let ylist = Object.keys(counter).sort();
	let year = ylist.length;
	let output = "<select class='year' onchange='doSearchConcerts()'>\n";
	output += `<option value="">Year [${year}]</option>`;
	for (let i=0; i<ylist.length; i++) {
		let name = ylist[i];
		let count = counter[ylist[i]];
		output += `<option value="${name}">${name} (${count})</option>`;
	}
	output += "</select>\n";
	return output;
}


//////////////////////////////
//
// buildProgramSourceSelect --
//

function buildProgramSourceSelect(data) {
	let counter = {};
	let sum = data.length;
	for (let i=0; i<sum; i++) {
		let entry = data[i];
		let programsource = entry[EMC.index.concerts.archive];
		if (!programsource) {
			//console.error("WARNING: ", entry, " DOES NOT HAVE AN PROGRAM SOURCE DESIGNATION");
			continue;
		}
		counter[programsource] = (counter[programsource] === undefined) ? 1 : counter[programsource] + 1;
	}

	let pslist = Object.keys(counter).sort();
	let programsource = pslist.length;
	let output = "<select class='programsource' onchange='doSearchConcerts()'>\n";
	output += `<option value="">Source of Program [${programsource}]</option>`;
	for (let i=0; i<pslist.length; i++) {
		let name = pslist[i];
		let count = counter[pslist[i]];
		output += `<option value="${name}">${name} (${count})</option>`;
	}
	output += "</select>\n";
	return output;
}


//////////////////////////////
//
// doSearchConcerts --
//

function doSearchConcerts(data) {
	if (!data) {
		data = EMC.METADATA.concerts;
	}
	console.error("input data for doSearchWorks", data);

	let searchInterface = EMC.menus.concerts;
	console.warn("print search interface", searchInterface);
	if (!searchInterface) {
		console.log("Problem finding search interface for concerts");
		return;
	}

	let countryField = searchInterface.querySelector("select.country");
	if (!countryField) {
		console.log("Problem finding country field in search interface");
		return;
	}
	let countryQuery = countryField.value;

	let yearField = searchInterface.querySelector("select.year");
	if (!yearField) {
		console.log("Problem finding year field in search interface");
		return;
	}
	let yearQuery = yearField.value;

	let programsourceField = searchInterface.querySelector("select.programsource");
	if (!programsourceField) {
		console.log("Problem finding country field in search interface");
		return;
	}
	let programsourceQuery = programsourceField.value;

	if (countryQuery !== "") {
		let tempdata = [];
		for (let i=0; i<data.length; i++) {
			let entry = data[i];
			let country = entry[EMC.index.concerts.country];
			if (country == countryQuery) {
				tempdata.push(entry);
			}
		}
		data = tempdata;
	}

	if (yearQuery !== "") {
		let tempdata = [];
		for (let i=0; i<data.length; i++) {
			let entry = data[i];
			let year = entry[EMC.index.concerts.year];
			if (year == yearQuery) {
				tempdata.push(entry);
			}
		}
		data = tempdata;
	}

	if (programsourceQuery !== "") {
		let tempdata = [];
		for (let i=0; i<data.length; i++) {
			let entry = data[i];
			let programsource = entry[EMC.index.concerts.archive];
			if (programsource == programsourceQuery) {
				tempdata.push(entry);
			}
		}
		data = tempdata;
	}

	displayBrowseTableConcerts(data);

}


//////////////////////////////
//
// doSearchWorks --
//

function doSearchWorks(data) {
	if (!data) {
		data = EMC.METADATA.works;
	}
	console.error("input data for doSearchWorks", data);


	let searchInterface = EMC.menus.works;
	console.warn("print search interface", searchInterface);
	if (!searchInterface) {
		console.log("Problem finding search interface for works");
		return;
	}

	let composerField = searchInterface.querySelector("select.composer");
	console.warn("print composerField", composerField);
	if (!composerField) {
		console.log("Problem finding composer field in search interface");
		return;
	}
	let composerQuery = composerField.value;

	let voiceField = searchInterface.querySelector("select.voice");
	if (!voiceField) {
		console.log("Problem finding voice-count field in search interface");
		return;
	}
	let voiceQuery = voiceField.value;

	let genreField = searchInterface.querySelector("select.genre");
	if (!genreField) {
		console.log("Problem finding genre field in search interface");
		return;
	}
	let genreQuery = genreField.value;

	let languageField = searchInterface.querySelector("select.language");
	if (!languageField) {
		console.log("Problem finding language field in search interface");
		return;
	}
	let languageQuery = languageField.value;

	let monopolyField = searchInterface.querySelector("select.monopoly");
	if (!monopolyField) {
		console.log("Problem finding monophonic/polyphonic field in search interface");
		return;
	}
	let monopolyQuery = monopolyField.value;

	let sacredsecularField = searchInterface.querySelector("select.sacredsecular");
	if (!sacredsecularField) {
		console.log("Problem finding sacred/secular field in search interface");
		return;
	}
	let sacredsecularQuery = sacredsecularField.value;

	let sourceField = searchInterface.querySelector("select.source");
	if (!sourceField) {
		console.log("Problem finding source field in search interface");
		return;
	}
	let sourceQuery = sourceField.value;

	let vocinstrField = searchInterface.querySelector("select.vocinstr");
	if (!vocinstrField) {
		console.log("Problem finding sacred/secular field in search interface");
		return;
	}
	let vocinstrQuery = vocinstrField.value;

	if (composerQuery) {
		let tempdata = [];
		for (let i=0; i<data.length; i++) {
			let entry = data[i];
			let composer = entry[EMC.index.works.composer];
			if (composer === composerQuery) {
				tempdata.push(entry);
			}
		}
		data = tempdata;
	}

	if (voiceQuery !== "") {
		let tempdata = [];
		for (let i=0; i<data.length; i++) {
			let entry = data[i];
			let voice = entry[EMC.index.works.voices];
			if (voice == voiceQuery) {
				tempdata.push(entry);
			}
		}
		data = tempdata;
	}

	if (genreQuery !== "") {
		let tempdata = [];
		for (let i=0; i<data.length; i++) {
			let entry = data[i];
			let genre = entry[EMC.index.works.genre];
			if (genre == genreQuery) {
				tempdata.push(entry);
			}
		}
		data = tempdata;
	}

	if (languageQuery !== "") {
		let tempdata = [];
		for (let i=0; i<data.length; i++) {
			let entry = data[i];
			let language = entry[EMC.index.works.language];
			if (language == languageQuery) {
				tempdata.push(entry);
			}
		}
		data = tempdata;
	}


	if (monopolyQuery !== "") {
		let tempdata = [];
		for (let i=0; i<data.length; i++) {
			let entry = data[i];
			let monopoly = entry[EMC.index.works.monopoly];
			if (monopoly == monopolyQuery) {
				tempdata.push(entry);
			}
		}
		data = tempdata;
	}

	if (sacredsecularQuery !== "") {
		let tempdata = [];
		for (let i=0; i<data.length; i++) {
			let entry = data[i];
			let sacredsecular = entry[EMC.index.works.sacrsec];
			if (sacredsecular == sacredsecularQuery) {
				tempdata.push(entry);
			}
		}
		data = tempdata;
	}

	if (sourceQuery !== "") {
		let tempdata = [];
		for (let i=0; i<data.length; i++) {
			let entry = data[i];
			let source = entry[EMC.index.works.source];
			if (source == sourceQuery) {
				tempdata.push(entry);
			}
		}
		data = tempdata;
	}

	if (vocinstrQuery !== "") {
		let tempdata = [];
		for (let i=0; i<data.length; i++) {
			let entry = data[i];
			let vocinstr = entry[EMC.index.works.vocinstr];
			if (vocinstr == vocinstrQuery) {
				tempdata.push(entry);
			}
		}
		data = tempdata;
	}

	displayBrowseTableWorks(data);
}

</script>
