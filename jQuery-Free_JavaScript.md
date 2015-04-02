JQuery-free JavaScript
======================

## When should I use jQuery ##

### When? ###
- when making prototypes of POCs
- when using jQuery plugins or dependent frameworks
- when you need support quirky browsers, like IE 8 or below

### Custom builds of jQuery ###
- Git pull the source and go to town
	+ Need git, nodejs, grunt, bower
- Use a jQuery builder service like manor.im/jquery-builder
- jQuery has focused modules that you can choose to include/exclude

### Alternate libraries ###
- Zepto.js - like a smaller jQuery, an option for mobile, doesn't support IE10 or below
- Min.js - *very* small and minimalist

### Demos ###
- Checkout caniuse.com for cross browser support
- Again, use nodejs, grunt, bower
- Demo of using these tools to build your project


## Native Selectors ##

Where you may have selectors like the following in jQuery:

```
$("#id");
$("tag");
$(".class");
$("input.date");
```

The native JS equivalents are

```
document.getElementById("id");
document.getElementsByTagName("tag");
document.getElementsByClassName("class"); // IE8 support?
document.getSelectorAll("input.date");
document.getSelector("input.date"); //returns the first match
```
Again, watch your browser support

## Each and looping ##

Using the above `document` methods return `node`'s, which need a little massaging to interate over. jQuery has a nice `$().each()` method to iterate over selectors. Here are some replacement code snippets

Opt 1
```
var i,
	nodes = document.querySelectorAll("input.date"),
	len = nodes.length;

for (i = 0; i < len; i++) {
	nodes[i].innerText = "Hello: " + i;
}
```
Using ECMAScript5's `forEach` method for arrays:

Opt 2
```
var nodes = document.querySelectorAll("input.date"),
	elems = Array.prototype.slice.call(nodes);

elems.forEach(function(element, index) {
	element.innerText = "hello: " + index;
});
```

Opt 3
```
var nodes = document.querySelectorAll("input.date");

[].forEach.call(nodes, function(element, index) {
	element.innerText = "hello: " + index;
});
```
Using a helper method
Opt 4
```
function $$(selector) {
	return [].slice.call(document.querySelectorAll(selector));
}

$$("input.date").forEach(function(element, index) {
	element.innerText = "hello: " + index;
}
```

### Index ###
jQuery
```
$("div.date").eq(3); //returns a jQuery wrapped element
$("div.date").get(3); //this and below return raw element
$("div.date")[3];
```

Native
```
document.querySelectorAll("div.date").item(3); //these of course are both the raw dom element
document.querySelectorAll("div.date")[3];
```

### First, Last ###

```
var dates = document.querySelectorAll("div.date");

//options for $().first()
dates[0];
//remember this returns the first match
document.querySelector("div.date");

//options for $().last()
dates[dates.length - 1];
[].pop.call(dates);
```
