---
layout: page
title: database
permalink: /database/
---

<div id="search-interface"></div>

<div id="list"></div>

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
	#search-interface { margin-bottom: 30px; }
	.wrapper {margin-left: 10px;}
	table.browse td:nth-child(2) {min-width: 125px;}
	table.browse td:nth-child(4) {white-space: nowrap;}
	table.browse td:nth-child(5) {min-width: 100px}
	table.browse td:nth-child(6) {min-width: 150px;}
	table.browse td:nth-child(7) {min-width: 200px;}
	select.source {max-width: 250px}
</style>

<script>
// vim: ts=3:nowrap

let METADATA = [];
let INDEX_name        	= "Standardized Name of Work";
let INDEX_composer    	= "Probable Composer";
let INDEX_voices       	= "Voices";
let INDEX_composername  = "Composer Name as Listed in Program";
let INDEX_conflattr     = "Conflicting Attributions";
let INDEX_language		= "Language";
let INDEX_language2		= "Second Language";
let INDEX_monopoly		= "Monophonic/Polyphonic";
let INDEX_sacrsec		= "Sacred/Secular";
let INDEX_vocinstr		= "Vocal/Instrumental";
let INDEX_genre			= "Genre";
let INDEX_source 		= "Source of Work Listed in Program";
let INDEX_folios		= "Folios/No.";
let INDEX_edition		= "Edition of Work Listed in Program";
let INDEX_pages			= "Nos./Page Numbers";
let INDEX_scanedition	= "Scan of Edition";
let INDEX_ProgID		= "Program ID";
let INDEX_ProgDate		= "Program Date";
let INDEX_ProgOrder		= "Order in Program";
let INDEX_NotesWork		= "Notes on Work";
let INDEX_ModernEd		= "Modern Edition";
let INDEX_Repeatcon		= "Repeat Concerts";

document.addEventListener("DOMContentLoaded", function () {
	var id = "AKfycbye6akuTA_UHOqrdZJwC9utsK1FTnUJIxTLndGibFgyNVPa0xW4FEsygSPfVsuHjsIsCg";
	var url = `https://script.google.com/macros/s/${id}/exec`;

	fetch(url)
	.then((response) => response.json())
	.then((data) => {
		processMetadata(data);
		buildSearchInterface(data, "#search-interface");
		displayBrowseTable(data, "#list"); 
	})
	.catch((error) => console.error("Error downloading metadata: ", error));

});

//////////////////////////////
//
// processMetadata --
//

function processMetadata(metadata) {

	let element = document.querySelector("#menu");
	if (!element) {
		console.error("Cannot find menu element");
		return;
	}

	let genres = {};
	for (let entry of metadata) {
		let genre = entry.INDEX_genre;
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
}


//////////////////////////////
//
// buildSearchInterface --
//

function buildSearchInterface(data, selector) {
	if (!selector) {
		selector = "#search-interface";
	}
	let element = document.querySelector(selector);
	if (!element) {
		console.error(`Error: cannot find ${selector} element to create search interface`);
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
// displayBrowseTable --
//

function displayBrowseTable(data, selector) {
	if (!selector) {
		selector = "#list";
	}
	let element = document.querySelector(selector);
	if (!element) {
		console.error(`Error: cannot find ${selector} element to display work table`);
		return;
	}
	let headings = [INDEX_name, INDEX_composer, INDEX_voices, INDEX_ProgDate, INDEX_genre, INDEX_source, INDEX_edition, INDEX_ModernEd];
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

			if (headings[i] == INDEX_edition) {
				let editioncombined = getEdition(entry);
				let url = getEditionUrl(entry);
				output += `<a target="_blank" href="${url}">${editioncombined}</a>`;
			} else if (headings[i] == INDEX_source){
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
		let composer = entry[INDEX_composer];
		if (!composer) {
			console.error("WARNING: ", entry, " DOES NOT HAVE A COMPOSER");
			continue;
		}
		counter[composer] = (counter[composer] === undefined) ? 1 : counter[composer] + 1;
	}

	let clist = Object.keys(counter).sort();
	let composerCount = clist.length;
	let output = "<select class='composer' onchange='doSearch()'>\n";
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
// buildGenreSelect --
//

function buildGenreSelect(data) {
	let counter = {};
	let sum = data.length;
	for (let i=0; i<sum; i++) {
		let entry = data[i];
		let genre = entry[INDEX_genre];
		if (!genre) {
			console.error("WARNING: ", entry, " DOES NOT HAVE A GENRE");
			continue;
		}
		counter[genre] = (counter[genre] === undefined) ? 1 : counter[genre] + 1;
	}

	let glist = Object.keys(counter).sort();
	let genreCount = glist.length;
	let output = "<select class='genre' onchange='doSearch()'>\n";
	output += `<option value="">Any genre [${genreCount}]</option>`;
	for (let i=0; i<glist.length; i++) {
		let name = glist[i];
		let count = counter[glist[i]];
		output += `<option value="${name}">${name} (${count})</option>`;
	}
	output += "</select>\n";
	return output;
}

//////////////////////////////
//
// getSource -- Generate Source + Folios/Pages 
//

function getSource(entry) {
	console.warn(entry);
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
	console.warn(entry);
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
		let language = entry[INDEX_language];
		if (!language) {
			console.error("WARNING: ", entry, " DOES NOT HAVE A LANGUAGE");
			continue;
		}
		counter[language] = (counter[language] === undefined) ? 1 : counter[language] + 1;
	}

	let llist = Object.keys(counter).sort();
	let languageCount = llist.length;
	let output = "<select class='language' onchange='doSearch()'>\n";
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
		let monopoly = entry[INDEX_monopoly];
		if (!monopoly) {
			console.error("WARNING: ", entry, " DOES NOT HAVE A MONOPHONIC/POLYPHONIC DESIGNATION");
			continue;
		}
		counter[monopoly] = (counter[monopoly] === undefined) ? 1 : counter[monopoly] + 1;
	}

	let mlist = Object.keys(counter).sort();
	let monopolyCount = mlist.length;
	let output = "<select class='monopoly' onchange='doSearch()'>\n";
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
		let sacredsecular = entry[INDEX_sacrsec];
		if (!sacredsecular) {
			console.error("WARNING: ", entry, " DOES NOT HAVE A SACRED/SECULAR DESIGNATION");
			continue;
		}
		counter[sacredsecular] = (counter[sacredsecular] === undefined) ? 1 : counter[sacredsecular] + 1;
	}

	let slist = Object.keys(counter).sort();
	let sacredsecularCount = slist.length;
	let output = "<select class='sacredsecular' onchange='doSearch()'>\n";
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
		let vocinstr = entry[INDEX_vocinstr];
		if (!vocinstr) {
			console.error("WARNING: ", entry, " DOES NOT HAVE A VOCAL/INSTRUMENTAL DESIGNATION");
			continue;
		}
		counter[vocinstr] = (counter[vocinstr] === undefined) ? 1 : counter[vocinstr] + 1;
	}

	let vilist = Object.keys(counter).sort();
	let vocinstrCount = vilist.length;
	let output = "<select class='vocinstr' onchange='doSearch()'>\n";
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
		let source = entry[INDEX_source];
		if (!source) {
			console.error("WARNING: ", entry, " DOES NOT HAVE A SOURCE");
			continue;
		}
		counter[source] = (counter[source] === undefined) ? 1 : counter[source] + 1;
	}

	let solist = Object.keys(counter).sort();
	let sourceCount = solist.length;
	let output = "<select class='source' onchange='doSearch()'>\n";
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
		let voice = entry[INDEX_voices];
		if (!voice) {
			console.error("WARNING: ", entry, " DOES NOT HAVE A VOICE COUNT");
			continue;
		}
		counter[voice] = (counter[voice] === undefined) ? 1 : counter[voice] + 1;
	}

	let vlist = Object.keys(counter).sort();
	let output = "<select class='voice' onchange='doSearch()'>\n";
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
// doSearch --
//

function doSearch(data) {
	if (!data) {
		data = METADATA;
	}

	let searchInterface = document.querySelector("#search-interface");
	if (!searchInterface) {
		console.log("Problem finding search interface");
		return;
	}

	let composerField = searchInterface.querySelector("select.composer");
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
			let composer = entry[INDEX_composer];
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
			let voice = entry[INDEX_voices];
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
			let genre = entry[INDEX_genre];
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
			let language = entry[INDEX_language];
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
			let monopoly = entry[INDEX_monopoly];
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
			let sacredsecular = entry[INDEX_sacrsec];
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
			let source = entry[INDEX_source];
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
			let vocinstr = entry[INDEX_vocinstr];
			if (vocinstr == vocinstrQuery) {
				tempdata.push(entry);
			}
		}
		data = tempdata;
	}


	displayBrowseTable(data);
}

</script>
