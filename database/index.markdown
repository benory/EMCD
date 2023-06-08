---
layout: page
title: database
---

<style>
	body {font: 400 12px/1 -apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,Helvetica,Arial,sans-serif,"Apple Color Emoji","Segoe UI Emoji","Segoe UI Symbol"}
	h1 { font-size: 40px; }
	th { text-align: left; }
	table.browse { min-width: 1000px;}
	table.browse { margin-left: auto; margin-right: auto; } /* center table */
	table.browse { border-collapse: collapse; } /* don't put gaps between cells */
	table.browse th { background:skyblue; }
	table.browse td, table.browse th {padding-left: 2px; padding-top: 2px; padding: 2px}
	table.browse tr:hover { background:#ff000011; }
	a { text-decoration: none; }
	span.browse-interface { margin-top: 30px; margin-bottom: 30px; }
	.wrapper {margin-left: 10px;}
	table.browse td:nth-child(2) {min-width: 125px;}
	table.browse td:nth-child(4) {white-space: nowrap;}
	table.browse td:nth-child(5) {min-width: 100px}
	table.browse td:nth-child(6) {min-width: 150px;}
	table.browse td:nth-child(7) {min-width: 200px;}
	select.source {max-width: 250px}
	span.sheet-button {
		display: inline-block;
		padding-bottom: 20px;
		padding-right: 20px;
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
EMC.METADATA = {};
EMC.METADATA.works = {% include_relative works.json %};
EMC.METADATA.concerts = {% include_relative concerts.json %};

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

EMC.index.concerts = EMC.index.works;

document.addEventListener("DOMContentLoaded", function () {
	buildSearchInterfaces(EMC.METADATA, "#browse-interface");
	displayBrowseTableWorks(EMC.METADATA.works);
});



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

function buildSearchInterfaceWorks(data, element) {
	if (!element) {
		console.error("ERROR: Cannot find search interface element", element);
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
	element.innerHTML = output;
}


//////////////////////////////
//
// buildSearchInterfaceConcerts --
//

function buildSearchInterfaceConcerts(data, browseElement) {
	let element = EMC.results.concerts;
	if (!element) {
		console.error("ERROR: Cannot find search interface element", element);
		return;
	}
	let output = "";
	//output += buildComposerSelect(data);
	//output += buildVoiceSelect(data);
	//output += buildGenreSelect(data);
	//output += buildLanguageSelect(data);
	//output += buildMonoPolySelect(data);
	//output += buildSacredSecularSelect(data);
	//output += buildVocInstrSelect(data);
	//output += buildSourceSelect(data);
	element.innerHTML = output;
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
	contents += "<table class='browse'>\n";
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

	let headings = [EMC.index.works.name, EMC.index.works.composer,
	EMC.index.works.voices, EMC.index.works.ProgDate,
	EMC.index.works.genre, EMC.index.works.source,
	EMC.index.works.edition, EMC.index.works.ModernEd];

	let contents = "";
	contents += "<table class='browse'>\n";
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
	let output = `<th>${headings.join("</th><th>")}</th>\n`;
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
			} else if (headings[i] == EMC.index.works.source){
				let sourcecombined = getSource(entry);
				output += sourcecombined;
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
// doSearchConcerts --
//

function doSearchConcerts(data) {

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
