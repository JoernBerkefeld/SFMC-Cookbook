# The SSJS Coding Guide

- [The SSJS Coding Guide](#the-ssjs-coding-guide)
	- [Style Guideline](#style-guideline)
	- [How to debug quickly](#how-to-debug-quickly)
	- [SSJS vs. JavaScript – the major differences that will break your code.](#ssjs-vs-javascript-%E2%80%93-the-major-differences-that-will-break-your-code)
		- [Standard JS features not available in SSJS](#standard-js-features-not-available-in-ssjs)
			- [“new” Operator](#new-operator)
			- [Returning a new object in a function](#returning-a-new-object-in-a-function)
			- [Using multiple script-tags and hoisting](#using-multiple-script-tags-and-hoisting)
		- [SSJS vs. SSJS documentation – stuff that simply does not work](#ssjs-vs-ssjs-documentation-%E2%80%93-stuff-that-simply-does-not-work)
			- [Retrieve()](#retrieve)

## Style Guideline

To get proper hints set your ESLint to "eslint:recommended", "google" in the “extends” settings attribute and the parserOptions.ecmaVersion to 3.
From there you need to add all globals (objects/classes that SSJS publishes into the global scope) and you are pretty much done. Please refer to the attached config for a complete setup.
In general, sticking to the official Angular.js 1.0 guide leads to good SSJS code as well.

**ATTACH .estlintrc FILE HERE**

**ATTACH package.json FILE HERE**

## How to debug quickly

Ever tried to fix a bug and got annoyed by having to publish your page or send your email 1000 times per day? Turns out you can load ANY text file that’s freely available on the internet AND run it through the server-side parsers for both AmpScript AND SSJS. The key is a method that’s similar to JavaScript’s eval() method together with another method that grabs the text content of a URL and returns it as a String:

```javascript
/* put this into an HTML block in your debug CloudPage (nothing else) */
%%= TreatAsContent(HTTPGet("https://myurl.com/loader.html")) =%%
```

```html
<!-- loader.html -->

<!-- optionally load AmpScript file -->
%%= TreatAsContent(HTTPGet("https://myurl.com/myampcode.amp")) =%%

<script runat="server" language="JavaScript">
	Platform.Load("core", "1.1.5"); // define one core version up front for all

	// load SSJS file #1 via AmpScript
	%%= TreatAsContent(HTTPGet("https://myurl.com/mycode1.ssjs")) =%%
	// load SSJS file #2 via AmpScript
	%%= TreatAsContent(HTTPGet("https://myurl.com/mycode2.ssjs")) =%%
	// load SSJS file #3 via AmpScript
	%%= TreatAsContent(HTTPGet("https://myurl.com/mycode3.ssjs")) =%%
</script>

<!-- optionally load HTML file via AmpScript -->
%%= TreatAsContent(HTTPGet("https://myurl.com/app.html")) =%%
```

```javascript
// mycode1.ssjs
function MyUtilities() {
    ...
}
```

```javascript
// mycode2.ssjs
function MySfmcApiWrapper() {
    ...
}
```

```javascript
// mycode3.ssjs
var util = MyUtilities();
var sfmc = MySfmcApiWrapper();

// ... fancy app code here
```

_Why?_ You save time because now you only update your online file on whatever webserver you like and don't need to re-publish your debug page ever again.

_How?_ Internally, AmpScript is always processed & executed first and only afterwards it runs through the SSJS processor.

## SSJS vs. JavaScript – the major differences that will break your code.

### Standard JS features not available in SSJS

This is a list of features that are standard in EcmaScript 3 but that are not supported in SSJS. I will also include a few operators & methods that only newer EcmaScript versions offer because I hear the question constantly

#### “new” Operator

Using “new” is common when you define your libraries in class format, however, this simply shows a 500 in SSJS. Writing the very same only without “new” works fine though.

```javascript
/* fails with 500 */
function MyClass() {
    var service = {
        attr1: true,
        method1: function(){…}
    };
    return service;
}
var myInstance = new MyClass();
myInstance.method1();
```

```javascript
/* good */
function MyClass() {
    var service = {
        attr1: true,
        method1: function(){…}
    };
    return service;
}
var myInstance = MyClass();
myInstance.method1();
```

#### Returning a new object in a function

Have you ever tried this returning an object as the result of a function? Well, better define it as a variable first and then return that variable, otherwise it causes a 500.

```javascript
/* fails with 500 */
function() {
    return {
        foo: bar
    };
}
```

```javascript
/* good */
function() {
    var temp = {
        foo: bar
    };
    return temp;
}
```

#### Using multiple script-tags and hoisting

Normally, you can easily spread your code over multiple files and as long as you are only referencing functions in your actual code, the functions can actually be placed at the end of your code and still be referenced up top. This principle is called hoisting and

```html
<!-- fails -->
<script runat="server" language="JavaScript">
	var myInstance = MyClass();
</script>

<script runat="server" language="JavaScript">
	function MyClass() {
	    var service = {
	        attr1: true,
	        method1: function(){…}
	    };
	    return service;
	}
</script>
```

```html
<!-- good -->
<script runat="server" language="JavaScript">
	var myInstance = MyClass();

	function MyClass() {
	    var service = {
	        attr1: true,
	        method1: function(){…}
	    };
	    return service;
	}
</script>
```

```html
<!-- good -->
<script runat="server" language="JavaScript">
	function MyClass() {
	    var service = {
	        attr1: true,
	        method1: function(){…}
	    };
	    return service;
	}
</script>

<script runat="server" language="JavaScript">
	var myInstance = MyClass();
</script>
```

### SSJS vs. SSJS documentation – stuff that simply does not work

Yup, this happens more often than you would think: The docs offer you this really nice method but for some reason it does not work at all, differently, or only in an automation but not in a cloudpage/email (or wise versa).

#### Retrieve()

This is a fun one because it comes with multiple challenges: It does not work AT ALL in CloudPages but it does work just fine in automations.
Also, if you leave out the filter parameter, which is officially an optional parameter, then it stops working all together. Also, if you try to omit it in a sneaky way by checking isNotNull on some field that is always filled it will still not work. You
