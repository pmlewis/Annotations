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

```
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

```
svg.selectAll("text")
  .data(yourData)
  .enter()
  .append("text")
  .text(function(d) { return d; })
  .attr("x", /*...*/)
  .attr("y", /* ... Remember to place things correctly ... */)
  .attr( { /* go to town on positioning, fonts, color, etc */ });
```

Building a line chart is a bit different, you have a bunch of points, and you'll connect each point sequentially. You'll use the `d3.svg.line` method to create a *line-drawing function*. You'll pass your data records into this *line-drawing function* and pass the value that spits out as a `d` property of a `path` element. You can follow `line` with `x` and `y` methods that'll define the x and y values for each data record. Then you can call `interpolate` to connect the dots.

```
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

```
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

