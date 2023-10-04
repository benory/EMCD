##
## Programmer:    Craig Stuart Sapp <craig@ccrma.stanford.edu>
## Creation Date: Thu Jun  8 19:45:08 PDT 2023
## Last Modified: Thu Jun  8 19:45:11 PDT 2023
## Syntax:        GNU Makefile
## Filename:      _includes/metadata/Makefile
## vim:           ts=3:nowrap
##
## Description: Download metadata for the 1520s project from the Google Spreadsheet
##              for the project's metadata.  You need to update the SID variable below
##              when the deployment of the Google Apps Script has changed.
##
##              Type "make" in this directory to download the JSON files for the metadata.
##

SID = AKfycbyCCOTXQ3kjJZOJPTXLAmseKlZSKS5p0zdY2JZcHVhwlubbQjNTyNlJKMRoH8J1IheRyQ

all: download

download: works concerts archives composers editions sources people bibliography

works:
	curl -L "https://script.google.com/macros/s/$(SID)/exec?sheet=works" > works.json

concerts:
	curl -L "https://script.google.com/macros/s/$(SID)/exec?sheet=concerts" > concerts.json

archives:
	curl -L "https://script.google.com/macros/s/$(SID)/exec?sheet=archives" > archives.json

composers:
	curl -L "https://script.google.com/macros/s/$(SID)/exec?sheet=composers" > composers.json

editions:
	curl -L "https://script.google.com/macros/s/$(SID)/exec?sheet=editions" > editions.json

sources:
	curl -L "https://script.google.com/macros/s/$(SID)/exec?sheet=sources" > sources.json

people:
	curl -L "https://script.google.com/macros/s/$(SID)/exec?sheet=people" > people.json

bibliography:
	curl -L "https://script.google.com/macros/s/$(SID)/exec?sheet=bibliography" > bibliography.json