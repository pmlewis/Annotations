# D3.js Data Visualization Fundamentals

D3 is a JavaScript library that helps you build data visualizations using formats native to the web. D3 helps you work with JSON, CSV, SVG, web APIs, the DOM, computing values, animations, etc.

SVG is a format for making scalable graphics, using shapes, filters, and gradients. D3 lets you create SVGs dynamically. SVGs use the coordinate system like most computer graphic formats use, upper left being 0,0 and going right or down increases the axis value.

Reference the `d3` object to use the D3 library. D3 uses a fluent function-chaining style for API calls.

`d3.select` function lets you target DOM elements a la jQuery using Selectors. `append(elementName)` adds an element as the next sibling of your `select`. `attr(attrName, attrValue)` adds an attribute, `style(styleProperty, styleValue)` adds the rule in the style attribute.

You can use `rect` and `circle` elements in an `svg` tag for basic shapes, and each have properties for defining the position and size of the shape. Your `svg` tag can have `width` and `height` properties...what happens when you don't define them? Do they default to full-width, height? 

You use `data()` and `enter()` to register your dataset to the selected elements, then if you use `append()`, d3 will apply that for each piece of data you pass in. If you use `attr()`, you can pass in a function as the second parameter that will map over each data value. That function will take value as 1st param, index as 2nd.


## Basic Charting ##

This will cover Bar charts, labels, line charts, and scatterplots.

For bar charts, remember you will append an svg, set a width and height on the svg, and you'll pass in an array of values. For example

```javascript
var svg = d3.select("body").append("svg").attr("width", yourWidth).attr("height", yourHeight);

svg.selectAll("rect")
  .data(yourData)
  .enter()
  .append("rect")
  .attr("x", function(d, i) { return i * (yourWidth / yourData.length); })
  .attr("y", function(d) { return yourHeight - (d); })
  .attr("width", yourWidth / yourData.length - barSpacing)
  .attr("height", function(d) { return (d); });
```

There are many svg attributes you can play with in code. `fill` lets you choose the color that fills a given shape. You can get real interesting when you work with functions that generate rgb (maybe rgba?) values.

The `attr` method can accept a object as a parameter where each property name is the attribute name you wish to create, and the value of each property gets generated for each data record you pass in.

You can now add some labels to your bars which, haha, is just a `text` svg element. If you wanted to add a text label for each data record you have, you could do

```javascript
svg.selectAll("text")
  .data(yourData)
  .enter()
  .append("text")
  .text(function(d) { return "Generate your text here"; })
  .attr("x", /*...*/)
  .attr("y", /* ... Remember to place things correctly ... */)
  .attr( { /* go to town on positioning, fonts, color, etc */ });
```

Building a line chart is a bit different, you have a bunch of points, and you'll connect each point sequentially. You'll use the `d3.svg.line` method to create a *line-drawing function*. You'll pass your data records into this *line-drawing function* and pass the value that spits out as a `d` property of a `path` element. You can follow `line` with `x` and `y` methods that'll define the x and y values for each data record. Then you can call `interpolate` to connect the dots.

```javascript
var myLineDrawingFunction = d3.svg.line()
	.x(function(d) { return d.month * 2; })
	.y(function(d) { return yourVizHeight - d.sales; }) // Remember to correctly orient yourself
	.interpolate("linear"); // There are different ways you can connect the dots

var myViz = mySelectedSvg.append("path") // Path elements are your lines in SVG
	.attr({
		d: myLineDrawingFunction(mySalesData),
		"stoke" : /* ... */
		"fill" : /* ... */
	});
```

You can also build Scatter Plots, which ends up just being placing `circle` elements for each data record you have.

```javascript
var dots = mySelectedSvg.selectAll("circle")
	.data(monthlySales)
	.enter()
	.append("circle")
	.attr({
		cx : function(d) { return d.month * myXConstant; },
		cy : function(d) { return mySvgHeight - d.sales; },
		r  : radiusConstant,
		"fill" : myDotColor
	});
```

Note that D3 has a lot of helper methods you can work with for doing data calculations, like `d3.max` and `d3.min`. Check out the docs. Remember that D3's strength is in being dynamic and interactive. You hook up interactive controls into your viz, and let users play with it. 


## External Data Sources - JSON and CSV ##

D3 can work with data found through API calls or by reading files, say, stored as CSV or JSON. CSV files are very common and serve as a good step before learning about using API calls.

Use the `d3.csv` method to work with a file path to your CSV and a function to work with the loaded dataset, like

```javascript
d3.csv('myDataset', function(error, loadedData) {
  if (error) { console.log(error); return; }
  // loadedData is all your data in an array of objects where each object has
  // properties named after your columns

  // Note the callback style of this method. csv() runs as async, so you'll
  // probably want to do your work drawing the viz inside this method
});
```

Working with the `d3.json` method is very similar to using`d3.csv`, except your dataset gets loaded as a JavaScript object, like you'd expect working with JSON.


## External Data Sources - Web APIs ##

D3 comes with functionality for accessing Web APIs. If you are working with JavaScript and you're referencing resources across different domains, or same domain different port, or same domain different protocol, you'll run into issues with the Same Origin Policy. This is configurable on the client browser and on servers. You really shouldn't mess around with this unless you're in Ops and know what you're doing, but a lot of public APIs are configured to allow cross origin requests.

The author shows that you can pass a URL as the first argument into `d3.json` since this particular URL returns a JSON response but the URL ends with a file extension, so I don't know if this is a shortcut for hitting APIs with GETs that return JSON or if this particular request is literally returning a file that is JSON formatted.

D3 does have `d3.xhr` that can work with AJAX requests if you need some more fine-grained control over the requests you make.

*Protip* - If you need to work with strings that are `base64` encoded, you can use `window.atob` to make it human readable. Also, if you need to make a string that is JSON into a real JavaScript object, use `JSON.parse`.

If you want some examples of open data APIs you can work with, check out data.gov, Twitter's API, and coinbase's bitcoin API. Read their documentation on how to use them, which may include learning how to authenticate with the API.


## Enhancing Your Viz with Scales and Axis ##

To make a scale, you need to pass in a range of values, then to get your scale, you are returned a different range of values. To easily make scales, you work with the `d3.scale` object. For now, you'll use `d3.scale.linear`.

```javascript
//To create a scaling function you specify the min and max
//domain value, then you specify a range to output to,
//which'll be the pixels you may draw
var scale = d3.scale.linear()
	.domain([salesMin, salesMax])
	.range([startLocation, endLocation]);

//Should be about halfway between start and end location
console.log(scale(salesAvg));
```

Once you build a scaling function, you can use this, say, in a line graph to generate your x or y values.

```javascript
var xScale = d3.scale.linear()
	.domain([
		d3.min(monthlySales, function(d) { return d.month; }),
		d3.max(monthlySales, function(d) { return d.month; })
	])
	.range(0, vizWidth);

var yScale = d3.scale.linear()
	// When making a line graph, your y values should most always
	// start at zero, otherwise you're probably "lying about your data"
	.domain([0, d3.max(monthlySales, function(d) { return d.sales; })])
	// Remember you'll flip the y axis since in SVG, increases in 
	// height grow down, so list the height first, then 0
	.range([vizHeight, 0]);

var drawLine = d3.svg.line()
	.x(function(d) { return xScale(d.month); })
	.y(function(d) { return yScale(d.sales); })
	.interpolate("linear");

// Now you can draw a scaled line graph. Try extending this to bar
// graphs and scatter plots!
```

Now to create an axis, you use the `d3.svg.axis` method. You pass in a scaling function, define an orientation, and then you call your axis drawing function. The author appends a `g` grouping element, then uses `call` passing in the scale function.

```javascript
var yAxisBuilder = d3.svg.axis().scale(yScale).orient("left");
var myYAxis = mySvg.append("g").call(yAxisBuilder)
	// Without this, the y-axis just looks like a line and you'll
	// be stuck scratching your head on what's wrong
	.attr("transform", "translate(" + shiftToRight +", 0)");

var xAxisBuilder = d3.svg.axis().scale(xScale).orient("bottom");
var myXAxis = mySvg.append("g").call(xAxisBuilder)
	// Similarly, you need to shift your x-axis "up"
	.attr("transform", "translate(0," + (vizHeight - shiftUpwards) + ")");
```

Note that if you inspect the axis created, you'll see a bunch of `g` elements with class `tick`. You can specify the number of ticks displayed(?) with the `tick` method on an `axis` object.

Note that we also created a x-axis and remember that time is often the unit of measure on on x-axis. D3 has a `time` object and that has a `scale` method also. `d3.time.scale().domain` can take JavaScript Date objects. After that, your axis builder function can use `tickFormat` with something like `xAxisBuilder.tickFormat(d3.time.format("%b"))` to short format your time dates. If you're doing any data tranformation, remember to tranform it also when drawing your viz, like in specifying your x values in a line drawing function.


## Enhancing your Viz with Interactivity ##

D3 comes with functionality for making your visualizations interactive.

### Letting the user select data to show ###

The author goes through a demo of creating a dropdown that will select which points in a scatterplot are labeled. We can use events and event listeners to control the showing and hiding.

D3 comes with some event listener methods similar to jQuery's.

```javascript
d3.select(".my-target")
	.on("change", function(d) {
		/* ... */
	});
```

The author similar grabs the `value` from the `<select>` on change, and redraws the text using `selectAll("text").data(myData).text` and passing a function that accepts the select value, and returns no text for the data that'll be hidden.

For redrawing an entire graph, note that you'll tend to not re-append your svg elements, you-ll use `selectAll` to grab the elements that already exist and just re-draw everything with the new data, or with the new event that shows up.

### Animations & Transitions ###

When you are selecting elements, and ultimately changing their positions, you can through in some transition methods before you change the position to add some animations. After you select, call `transition`, and follow it up if you'd like with calls to say `duration` and `ease`.

### Tooltips ###

For elements, you can add an HTML title with the `title` method built into D3 that displays the value on hover, and this is built into browsers. This is pretty basic though. The author shows that you can just add `mouseover` events to draw whatever you want with the same chain that builds, say, your bars in a bar graph, then remove what you draw on `mouseout`. The author shows that you can add some text blocks on hover with a transition in and an opacity for a nice effect.


## Mapping ##

D3 can let you draw maps(!) and add features to the maps, like color or bordering things off. A *choropleth* is a map that has been sectioned off and patterned to illustrate statistical information. D3 can help you make choropleths.

D3 uses GeoJSON as a format for drawing shapes, like each map shape, country shape, county shape, etc.

census.gov has a section for geography and map data that you can use to grab cartographic boundary data which can be converted into GeoJSON files. There should be more information on the process of converting different kinds of shape files into GeoJSON in D3's documentation, but long story short, use tools to handle the conversions for you.

To draw geographic shapes, you'll use the `d3.geo` object which has a `path` method. The way you can get started drawing a basic map is to create a basic geo path function, read in some json containing the map data, set that json as data, then append a path setting the `d` attribute as your path building function.

```javascript
var geoPathBuilder = d3.geo.path();
// We have grabbed our GeoJSON for the United States and it's living in the same dir
d3.json("us.json", function(myMapJson) {
	
	mySvg.selectAll("path")
		.data(myMapJson.features)
		.enter()
		.append("path")
		.attr("d", geoPathBuilder)
		.attr("fill", myMapColor);
});
```

That's cool, and it draws the US, but the size is pretty fixed. If you want to change the size of the map, you'll have to use a *projection* function to basically calculate what the map should look like for a given scale and size.

```javascript
// Albers projection is the default projection used for the US
// map we are using.
var myProjection = d3.geo.albersUsa()
	.translate([svgWidth/2, svgHeight/2])
	.scale([svgWidth]);

geoPathBuilder = d3.geo.path().projection(myProjection);

// Continue on like above
```

To continue making your choropleth, you need some data, and you'll need a means to draw color for boundaries, so we'll use the US map and a csv of sales data per state.

Load the sales data and the map json. We can use `d3.scale` and pass into `range` an array of colors, like strings such as `"rgb(254, 240, 217)"`. Using a scaling function using a range of discrete values should handle fitting your data into what category it should be in. Once your data is loaded, you can set the domain as 0 to your maximum data value to the scale building function.
 
The author joins the data from the csv to the json that came back from the map data based on the state's name. Then you can use your color building function to set the fill color of each data when you're appending the path.

```javascript
mySvg.selectAll("path")
	// We have joined our sales data to features[*].properties.value
	.data(myUsaJson.features)
	.enter()
	.append("path")
	.attr("d", geoPathBuilder)
	.style("fill", function(d) {
		var salesVal = d.properties.value;
		if (value) {
			return myColorScale(salesVal);
		} else {
			return someDefaultColorOrNothing;
		}
	});
```

Suppose you want to draw points on a map. You need position data for each point, and in the case of cities in a country, latitude and longitude are readily available. In the case of cities on the US map, our projection function (what we created using `albersUsa()` can accept a long, lat pair as an array and returns an array with long in index 0, lat in index 1. How can we do this though for maps we ourselves create?

```javascript
mySvg.selectAll("circle")
	.data(salesByCityData)
	.enter()
	.append("circle")
	.attr({
		cx  : function(d) { return myProjection([d.long, d.lat])[0]; },
		cy  : function(d) { return myProjection([d.long, d.lat])[1]; },
		r   : function(d) { return sizeProportion(d.sales); },
		fill: myColor
	});
```