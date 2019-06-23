# The SSJS Coding Guide

- [The SSJS Coding Guide](#the-ssjs-coding-guide)
	- [Style Guideline](#style-guideline)
	- [SSJS snippets](#ssjs-snippets)
		- [Reading GET parameters](#reading-get-parameters)
		- [Reading POST form parameters](#reading-post-form-parameters)
		- [Reading POST payload](#reading-post-payload)
		- [Getting/Setting AMPscript variables](#gettingsetting-ampscript-variables)
		- [Get/Set Cookies](#getset-cookies)
		- [Update (All) Subscriber List](#update-all-subscriber-list)
		- [Data Extensions](#data-extensions)
			- [SELECT](#select)
			- [INSERT, UPSERT, UPDATE, DELETE](#insert-upsert-update-delete)
	- [SSJS vs. JavaScript – the major differences that will break your code.](#ssjs-vs-javascript--the-major-differences-that-will-break-your-code)
		- [Standard JS features not available in SSJS](#standard-js-features-not-available-in-ssjs)
			- [“new” Operator](#new-operator)
			- [Returning a new object in a function](#returning-a-new-object-in-a-function)
			- [Using multiple script-tags and hoisting](#using-multiple-script-tags-and-hoisting)
		- [SSJS vs. SSJS documentation – stuff that simply does not work](#ssjs-vs-ssjs-documentation--stuff-that-simply-does-not-work)
			- [Retrieve()](#retrieve)
			- [Platform.Request.GetPostData()](#platformrequestgetpostdata)
			- [DateExtension.Init(): DE Name vs. External Key](#dateextensioninit-de-name-vs-external-key)
			- [switch - default](#switch---default)

## Style Guideline

To get proper hints set your ESLint to "eslint:recommended", "google" in the “extends” settings attribute and the parserOptions.ecmaVersion to 3.
From there you need to add all globals (objects/classes that SSJS publishes into the global scope) and you are pretty much done. Please refer to the attached config for a complete setup.
In general, sticking to the [official Angular.js 1.0 guide](https://github.com/johnpapa/angular-styleguide/blob/master/a1/README.md) leads to good SSJS code as well.

Download these for general setup:

-   [.eslintrc](../.eslintrc)
-   [.prettierrc](../.prettierrc)
-   [.editorconfig](../.editorconfig)
-   [package.json](../package.json)

And these if you will use Visual Studio Code (place them .vscode folder inside the root of your project folder):

-   [Recommended VSC Extensions](../.vscode/extensions.json)
-   [Recommended VSC Settings](../.vscode/settings.json)

## SSJS snippets

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

### Get/Set Cookies

To read cookies, all you need to know is the name.

_Docs [Request.GetCookieValue](https://developer.salesforce.com/docs/atlas.en-us.mc-programmatic-content.meta/mc-programmatic-content/ssjs_platformHTTPPropertyGetCookieValue.htm)_

```javascript
/* getter */
var oldCookieValue = Platform.Request.GetCookieValue('cookie_name');
```

When setting a cookie you may _optionally_ specifiy (a) the `Expires` date & time and (b) if the cookie value shall only be shared with the server if the client called it via https (`Secure` attribute).

Other standard attributes are not supported, which also excludes `HTTP1.1`'s `Max-Age`, rendering this implementaion deprecated by modern standards. So far it's still working though as browsers didn't drop support yet.

The official docs state a weird date format (`"2015-12-31 140000.999"`) but in reality a more standard format like `"5/1/2019 12:12:06"` works fine. SFMC will autoconvert it from its internal timezone to GMT, making it compatible with the standard for cookies.

_Docs [Response.SetCookie](https://developer.salesforce.com/docs/atlas.en-us.mc-programmatic-content.meta/mc-programmatic-content/ssjs_platformClientBrowserSetCookie.htm)_

```javascript
/* setter */
var newCookieValue = 'foo bar';
var expirationString = getTimestamp(new Date()); // optional parameter
var httpsOnly = true; // optional parameter
Platform.Response.SetCookie('cookie_name', newCookieValue, expirationString, httpsOnly);

/* helper function that returns the date in a format SFMC can understand */
function getTimestamp(dt) {
	var h = dt.getHours();
	var m = dt.getMinutes();
	var s = dt.getSeconds();
	if (h < 10) {
		h = '0' + h;
	}
	if (m < 10) {
		m = '0' + m;
	}
	if (s < 10) {
		s = '0' + s;
	}
	var ts = dt.getMonth() + 1 + '/' + dt.getDate() + '/' + dt.getFullYear() + ' ' + h + ':' + m + ':' + s;

	return ts;
}
```

### Update (All) Subscriber List

First, you need to know the ID of the List you want to update. Even the All Subscriber List has it's own id and it's not necessarily "1". You can get that in the Property page of the list:

1. Go to Email Studio
2. In the top navation click on Subscribers -> Lists
3. In the left navigation, click on the first link, `My Subscribers`
4. Tick the checkbox in front of `All Subscribers`, then click on `Properties`
5. Here under List Identification > List Attributes you will find a table that has the required `ID` as first column.

```javascript
// *** preparation ***

var newStatus = 'Unsubscribed'; // Active|Unsubscribed
var subscriberListID = 12; // see how to find the ID above; potentially different on every instance!
var mySubKey = 'test@domain.com'; // this is the subscriberKey of the contact that you want to udpate
var subscriberObj = {
	Attributes: {}, // optional custom attributes
	EmailTypePreference: 'HTML',
	Lists: {
		Status: newStatus,
		ID: subscriberListID,
		Action: 'Update'
	},
	SubscriberKey: mySubKey
};

// optionally add custom attribues:
subscriberObj.Attributes = {
	foo: 'bar',
	hello: 'world'
};

// *** execute update ***

// instantiate your subscriber
var mySubscriber = Subscriber.Init(mySubKey);
// run the update
var subscriptionStatus = mySubscriber.Update(subscriberObj);
```

### Data Extensions

There are multiple types of methods in SSJS for the same purpose but each with their own drawbacks and benefits:

|                  | `XxxxData(...)`                                                                                                                       | `XxxxDE(...)`                                             | `Rows.Xxxx(...)` |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------- | ---------------- |
| **When to use**  | - **landing pages** (°°°)<br>- SMS & MMS in MobileConnect (°)<br>- push messages in MobilePush (°)<br>- messages in GroupConnect. (°) | - landing pages (°°)<br>- email messages at send time (°) | ?                |
| **return value** | `{integer}` number of affected rows                                                                                                   | `void`                                                    | ?                |

Legend:

-   **bold** : recommended
-   (°) : according to docs (TODO: test this)
-   (°°) : not in docs but working
-   (°°°) : according to docs & confirmed

The code samples are identical for `XxxxData()` and `XxxxDE()` aside from the option the check the number of affected rows.

**Note:** There is no option to filter (WHERE) with OR combinations. Only AND combos are possible. When function parameters are arrays, you may pass in one or multiple values. Just make sure that column names and column values are both provided.

#### SELECT

```javascript
// *** SELECT ***
// Having 1 where condition is obligatory.
// Hack: Add a column that always has the same value if you want to be able to get all rows.

// config
var deName = 'name_of_your_de'; // REQUIRED
var whereColumnArr = ['where_col1', 'where_col2']; // REQUIRED
var whereColumValueArr = ['value1', 'value2']; // REQUIRED
var limit = 50; // REQUIRED
var orderBy = 'first name'; // OPTIONAL; omit parameter if no sorting needed

// execution
var myDe = DataExtension.Init(deName); // add 'ENT.' in front of the name for shared DEs!
var myDeArr = myDe.Rows.Lookup(whereColumnArr, whereColumValueArr, limit, orderBy);

// parsing
// if no rows where returned, myDeArr returns NULL
// optionally normalize the result:
if (myDeArr === null) {
	myDeArr = [];
}

if (myDeArr.length) {
	// your code here
}
```

```javascript
// TODO Lookup: https://developer.salesforce.com/docs/atlas.en-us.mc-programmatic-content.meta/mc-programmatic-content/ssjs_platformDataExtensionLookup.htm

// TODO LookupRows: https://developer.salesforce.com/docs/atlas.en-us.mc-programmatic-content.meta/mc-programmatic-content/ssjs_platformDataExtensionLookupRows.htm

// TODO LookupOrderedRows: https://developer.salesforce.com/docs/atlas.en-us.mc-programmatic-content.meta/mc-programmatic-content/ssjs_platformDataExtensionLookupOrderedRows.htm
```

```javascript
// TODO test & explain Rows.Retrieve() compared to Rows.Lookup()
// https://developer.salesforce.com/docs/atlas.en-us.mc-programmatic-content.meta/mc-programmatic-content/ssjs_dataExtensionRowsRetrieve.htm

// already known that this works in automation but NOT in landing pages
// it does NOT work WITHOUT the filter, despite the docs stating the contrary
var birthdayDE = DataExtension.Init('birthdayDE');
// according to the docs this filter allows complex structures including OR
var filter = { Property: 'Age', SimpleOperator: 'greaterThan', Value: 20 };
var data = birthdayDE.Rows.Retrieve(filter);
```

#### INSERT, UPSERT, UPDATE, DELETE

```javascript
// *** INSERT ***
// the docs propose to add 2 parameters per column/value combination instead of 2 arrays.
// --> ignore the docs.

var deName = 'myDe'; // REQUIRED
var insertColumnArr = ['lastname', 'age']; // REQUIRED
var insertColumnValueArr = ['Connor', '16']; // REQUIRED

var insertedRowCount = Platform.Function.InsertData(deName, insertColumnArr, insertColumnValueArr);
// or
Platform.Function.InsertDE(deName, insertColumnArr, insertColumnValueArr);
```

```javascript
// TODO test & compare Rows.Add()
// https://developer.salesforce.com/docs/atlas.en-us.mc-programmatic-content.meta/mc-programmatic-content/ssjs_dataExtensionRowsAdd.htm

var arrContacts = [
	{ Email: 'jdoe@example.com', FirstName: 'John', LastName: 'Doe' },
	{ Email: 'aruiz@example.com', FirstName: 'Angel', LastName: 'Ruiz' }
];

var birthdayDE = DataExtension.Init('birthdayDE');
birthdayDE.Rows.Add(arrContacts);
```

```javascript
// *** UPSERT ***
// the docs propose to pass in strings instead of arrays
// --> ignore the docs.

var deName = 'myDe'; // REQUIRED
var whereColumnArr = ['firstname']; // REQUIRED
var whereColumValueArr = ['John']; // REQUIRED
var upsertColumnArr = ['lastname', 'age']; // REQUIRED
var upsertColumnValueArr = ['Connor', '16']; // REQUIRED

var upsertedRowCount = Platform.Function.UpsertData(
	deName,
	whereColumnArr,
	whereColumValueArr,
	upsertColumnArr,
	upsertColumnValueArr
);
// or
Platform.Function.UpsertDE(deName, whereColumnArr, whereColumValueArr, upsertColumnArr, upsertColumnValueArr);
```

```javascript
// *** UPDATE ***
// the docs propose to pass in strings instead of arrays
// --> ignore the docs.

var deName = 'myDe'; // REQUIRED
var whereColumnArr = ['firstname']; // REQUIRED
var whereColumValueArr = ['John']; // REQUIRED
var upsertColumnArr = ['lastname', 'age']; // REQUIRED
var upsertColumnValueArr = ['Connor', '16']; // REQUIRED

var updatedRowCount = Platform.Function.UpdateData(
	deName,
	whereColumnArr,
	whereColumValueArr,
	upsertColumnArr,
	upsertColumnValueArr
);
// or
Platform.Function.UpdateDE(deName, whereColumnArr, whereColumValueArr, upsertColumnArr, upsertColumnValueArr);
```

```javascript
// TODO test & compare Rows.Update()
// https://developer.salesforce.com/docs/atlas.en-us.mc-programmatic-content.meta/mc-programmatic-content/ssjs_dataExtensionRowsUpdate.htm

var birthdayDE = DataExtension.Init('birthdayDE');
birthdayDE.Rows.Update({ Age: '25' }, ['FirstName'], ['Angel']);
```

```javascript
// *** DELETE ***
// the docs propose to pass in strings instead of arrays
// --> ignore the docs.

var deName = 'myDe'; // REQUIRED
var whereColumnArr = ['firstname']; // REQUIRED
var whereColumValueArr = ['John']; // REQUIRED

var deletedRowCount = Platform.Function.DeleteData(deName, whereColumnArr, whereColumValueArr);
// or
Platform.Function.DeleteDE(deName, whereColumnArr, whereColumValueArr);
```

```javascript
// TODO test & compare Rows.Remove()
// https://developer.salesforce.com/docs/atlas.en-us.mc-programmatic-content.meta/mc-programmatic-content/ssjs_dataExtensionRowsRemove.htm

var birthdayDE = DataExtension.Init('birthdayDE');
birthdayDE.Rows.Remove(['FirstName', 'Age'], ['Angel', 24]);
```

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

#### Platform.Request.GetPostData()

This only works the first time you call it. Every next execution returns nothing.

#### DateExtension.Init(): DE Name vs. External Key

In Automations you NEED to use the External Key. Meanwhile in Landing pages, E-Mails and SMS you can use the Name or the External Key.
--> Always set your External Key to the same value as the name to circumvent this problem.

#### switch - default

Using a normal switch statement with a `default` part at the end can lead to that section not being processed. Avoid `default` and instead set
