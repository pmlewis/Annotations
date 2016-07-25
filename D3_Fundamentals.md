# D3.js Data Visualization Fundamentals

D3 is a JavaScript library that helps you build data visualizations using formats native to the web. D3 helps you work with JSON, CSV, SVG, web APIs, the DOM, etc.

SVGs are is a format for making scalable graphics, using shapes, filters, and gradients. D3 lets you create SVGs dynamically. SVGs use the coordinate system like most computer graphic formats use, upper left being 0,0 and going right or down increases the axis value.

Reference the `d3` object to use the D3 library. D3 uses a fluent function-chaining style for API calls.

`d3.select` function lets you target DOM elements a la jQuery using Selectors. `append(elementName)` adds an element as the next sibling of your `select`. `attr(attrName, attrValue)` adds an attribute, `style(styleProperty, styleValue)` adds the select element.

You can use `rect` and `circle` elements in an `svg` tag for basic shapes, and each have properties for defining the position and size of the shape. Your `svg` tag can have `width` and `height` properties...what happens when you don't define them? Do they default to full-width, height? 

You use `data()` and `enter()` to register your dataset to the selected elements, then if you use `append()`, d3 will apply that for each piece of data you pass in. If you use `attr()`, you can pass in a function as the second value that will map over each data value. That function will take value as 1st param, index as 2nd.
