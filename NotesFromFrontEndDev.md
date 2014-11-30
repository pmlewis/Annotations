# Front End Web Dev

## HTTP and Server interactions
* Http requests
	* Req -> res
		* Get - itself has not body, but res will (read)
		* Put - has body, res may have body, may change content (create)
		* Post - used when sumbiting data, always has a body (edit)
		* Delete - no body, res has no body (delete)
	* cookies - sent with requests and responses
* XHR - Ajax calls
	* Javascript making reqs, uses the same above cmds
	* jquery - `$.get(...) $.post, etc`
* JSON
	* not assigned to vars
	* keys must be strings, have quotes
	* No functions to keys
			 "course" : {
				"name": "pluralsight"
				}
	* `var json = JSON.stringify(someobj); var out = JSON.parse(json);`
* Page request lifecycle
	* typically, the first thing to come back from a get req for a html page is the html itself, then the doc will get parsed and start making reqs for the content linked to by the doc, like sytlesheets, scripts, images

## Browser
* evergreen vs nonevergreen
	* auto updating? nonevergreen: IE
* versions of features supported by each browser
	* caniuse.com
	* dealing with unsupported features
		* fallbacks
			* secondary choice - detect cant use, remove element, replace with something else
		* polyfill
			* add some special sauce to get your thing working in the unsupported browser, like a third party lib
		* Progressive Enhancement
			* start from the bottom, minimum fetures, then build from there
* Dev tools in browser
	* Chrome's
		* check out pluralsight's course on chrome's dev tools
		* ctrl shift c - inspect element
		* sources, ctrl o to find files
	* firefox's
* Automated testing in other browsers
	* Visual tests and non visual tests
	* nonvisual
		* karma - built by the angular.js people
