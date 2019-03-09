# The SSJS Coding Guide

- [The SSJS Coding Guide](#the-ssjs-coding-guide)
	- [Style Guideline](#style-guideline)
	- [SSJS snippets for typical problems](#ssjs-snippets-for-typical-problems)
		- [Reading GET parameters](#reading-get-parameters)
		- [Reading POST form parameters](#reading-post-form-parameters)
		- [Reading POST payload](#reading-post-payload)
		- [Getting/Setting AMPscript variables](#gettingsetting-ampscript-variables)
	- [SSJS vs. JavaScript – the major differences that will break your code](#ssjs-vs-javascript--the-major-differences-that-will-break-your-code)
		- [Standard JS features not available in SSJS](#standard-js-features-not-available-in-ssjs)
			- [“new” Operator](#new-operator)
			- [Returning a new object in a function](#returning-a-new-object-in-a-function)
			- [Using multiple script-tags and hoisting](#using-multiple-script-tags-and-hoisting)
		- [SSJS vs. SSJS documentation – stuff that simply does not work](#ssjs-vs-ssjs-documentation--stuff-that-simply-does-not-work)
			- [Retrieve()](#retrieve)
			- [Platform.Request.GetPostData()](#platformrequestgetpostdata)

## Style Guideline

To get proper hints set your ESLint to "eslint:recommended", "google" in the “extends” settings attribute and the parserOptions.ecmaVersion to 3.
From there you need to add all globals (objects/classes that SSJS publishes into the global scope) and you are pretty much done. Please refer to the attached config for a complete setup.
In general, sticking to the [official Angular.js 1.0 guide](https://github.com/johnpapa/angular-styleguide/blob/master/a1/README.md) leads to good SSJS code as well.

Download these for general setup:

- [.eslintrc](../.eslintrc)
- [.prettierrc](../.prettierrc)
- [.editorconfig](../.editorconfig)
- [package.json](../package.json)

And these if you will use Visual Studio Code (place them .vscode folder inside the root of your project folder):

- [Recommended VSC Extensions](../.vscode/extensions.json)
- [Recommended VSC Settings](../.vscode/settings.json)

## SSJS snippets for typical problems

### Reading GET parameters

GET parameters require you to know the name. It seems there is no way to get all parameters or the query string itself.

```html
<!-- GET url.com/cloudpage?data=test -->
<script runat="server" language="JavaScript">
Platform.Load("core", "1.1.1");

var param = Platform.Request.GetQueryStringParameter("data");
// param now has the value "test"
</script>
```

_Note: it also works if you omit the `Platform.` in front of `Request.`_

### Reading POST form parameters

This is a shortcut if you want to read only a few values. Below is an option to retrieve all form values at once.

```html
<!-- GET url.com/cloudpage with form field "firstname" -->
<script runat="server" language="JavaScript">
Platform.Load("core", "1.1.1");

var param = Platform.Request.GetFormField("firstname");
// param now has the value "firstname"
</script>
```

### Reading POST payload

```html
<!-- POST url.com/cloudpage -->
<script runat="server" language="JavaScript">
Platform.Load("core", "1.1.1");

var postBody = Platform.Request.GetPostData();
</script>
```

**Note:** This method only works the first time you call it. Every next execution returns nothing. Save the response in a variable if you need it multiple times.

The return value varies depending on the header. Some bad payload-header combinations will cause immediate 500 errors while others work fine.

**Allowed Content-Type:**

| Header                                                             | Sample payload                                                                   | Sample return                   | Comment                             |
| ------------------------------------------------------------------ | -------------------------------------------------------------------------------- | ------------------------------- | ----------------------------------- |
| `application/x-www-form-urlencoded`                                | `firstname=John&lastname=Smith`                                                  | `firstname=John&lastname=Smith` | this is used for normal form fields |
| `application/json` / `application/xml` / `text/xml` / `text/plain` | `whatever you like`; providing a JSON value allows you to later parse it as such | `whatever you like`             |

```html
<!-- enhanced wrapper to deal with post data -->
<!-- post body: {"attribute1": "test", "foo":"bar"} -->

<script runat="server" language="JavaScript">
Platform.Load("core", "1.1.1");

function PostBody() {
	var postBody = null;
	var service = {
		json: json,
		form: form,
		text: text
	};

	// init
	_init();

	return service;

	function _init() {
		postBody = Platform.Request.GetPostData();
	}

	function text() {
		return postBody;
	}
	function json() {
		var temp = Platform.Function.ParseJSON(postBody);
		if (!temp && postBody) {
			temp = {error: "input was no string and not JSON formatted"};
		}
		return temp;
	}
	function form() {
		var obj = {};
		var tempArr = postBody.split("&");
		for (var i = 0; i < tempArr.length; i++) {
			var temp = tempArr[i].split("=");
			if(temp.lengh>1) {
				obj[temp[0]] = temp[1];
			}
		}
		return obj;
	}
}

// demo on how to use it
var myPostBody = PostBody();
Write(myPostBody.text() + "\n");
// {"attribute1": "test", "foo":"bar"}

var json = myPostBody.json();
for (var el in json) {
	Write(el + ": " + json[el] + "\n");
	// attribute1: test
	// foo: bar
}

var form = myPostBody.form();
for (var el in form) {
	Write(el + ": " + form[el] + "\n");
	// attribute1: test
	// foo: bar
}

</script>

```

### Getting/Setting AMPscript variables

```html
<!-- setter.amp -->
%%[

Set @myAmpscriptVariable = "test"

]%%

<!-- getter.ssjs -->
<script runat="server" language="JavaScript">
Platform.Load("core", "1.1.1");

var temp = Variable.GetValue('@myAmpscriptVariable');
// temp now has the value "test"
</script>
```

```html
<!-- setter.ssjs -->
<script runat="server" language="JavaScript">
Platform.Load("core", "1.1.1");

var temp = 'test2';
Variable.SetValue('@myAmpscriptVariable', temp);
// @myAmpscriptVariable now has the value "test2"
</script>

<!-- getter.amp -->
%%[

Set @testVariable = @myAmpscriptVariable

]%%
<!-- @testVariable is now set to "test2"; one can of course you -->
<!-- the following outputs "test2" -->
%%= var(@myAmpscriptVariable) =%%
```

## SSJS vs. JavaScript – the major differences that will break your code

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

#### Platform.Request.GetPostData()

This only works the first time you call it. Every next execution returns nothing.