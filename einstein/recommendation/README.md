# Einstein Recommendations <!-- omit in toc -->

This page aims to make using Einstein recommendations a little easier by adding a few explanations around how to use it in a modern setup.

- [Collect Code](#collect-code)
	- [Note to developers](#note-to-developers)
	- [Collect Code on page load](#collect-code-on-page-load)
	- [Asynchronous Collect Code](#asynchronous-collect-code)
	- [Asynchronous Collect Code with preloading](#asynchronous-collect-code-with-preloading)

## Collect Code

> Official documentation: [help.salesforce.com](https://help.salesforce.com/articleView?id=mc_ctc_collect_code.htm&type=5)

The default sample codes all assume a page load is happening whenever something needs to be tracked. However, the `collect.js` library is actually capable of being loaded asynchronously and can be used in a single-page-application (SPA) environment as well, similarly to [Google's analytics.js](https://developers.google.com/analytics/devguides/collection/analyticsjs).

### Note to developers

While the offical sample code snippets might lead you to believe that you are looking at an **Array**, that is in fact only half true. The variable `_etmc` gets rewritten by `collect.js` to become an object that happens to also expose a method named `push()`. Their base snippet in fact assumes that `collect.js` has been loaded synchronously before you execute your first call to `push()`. Essentially, the first Array-element is the name of an internal method of collect.js that gets executed.

For more details, have a look at the [collect.js](collect-code/default/collect.js) library itself.

### Collect Code on page load

_Quoting from [SFMC's documentation](https://help.salesforce.com/articleView?id=mc_ctc_install_collect_code.htm&type=5):_

The Collect Code should be placed just before the closing head tag and before any other Marketing Cloud code.

To initialize the tracking, do not forget to replace the two occurences of "INSERT_MID" with your actual BU's Member ID.

```html
<script src='//INSERT_MID.collect.igodigital.com/collect.js'></script>
```

This code should be placed directly below the `</head>` tag following the above tag.

```html
<script>
  _etmc.push(['setOrgId', 'INSERT_MID']);
  _etmc.push(['trackPageView']);
</script>
```

Download: [Default sample code](collect-code/default/collect-code.defaut.html)

### Asynchronous Collect Code

The Collect Code tag should be added near the top of the `<head>` tag and before any other script or CSS tags.

To initialize the tracking, do not forget to replace the two occurences of "INSERT_MID" with your actual BU's Member ID.

```html
<!-- Einstein Collect Code -->
<script>
(function(e,t,c,n,o,s,a){e[o]=e[o]||[],s=t.createElement(c),a=t.getElementsByTagName(c)[0],s.async=1,s.src=n,a.parentNode.insertBefore(s,a)})(window,document,'script','//INSERT_MID.collect.igodigital.com/collect.js','_etmc');

// always run this line once, followed by what you actually want to track; can be run programmatically in single-page-applications
_etmc.push(['setOrgId', 'INSERT_MID']);

// this can be run programmatically if your website is a single-page-application
_etmc.push(['trackPageView']);
</script>
<!-- End Einstein Collect Code -->
```

Download: [Async sample code](collect-code/collect-code.async.html)

The above code does four main things:

1. Creates a `<script>` element that starts asynchronously downloading the collect.js JavaScript library from `https://INSERT_MID.collect.igodigital.com/collect.js`
2. Initializes a global `_etmc` function that allows you to schedule commands to be run once the collect.js library is loaded and ready to go.
3. Adds a command to the `_etmc` command queue to _set the Org ID_ to your Business Unit Member ID. This is required to ensure your tracking data ends up in the correct Business Unit.
4. Adds another command to the `_etmc` command queue to track a [pageview](https://help.salesforce.com/articleView?id=mc_ctc_track_page_view.htm&type=5) to Einstein for the current page.

Custom implementations may require modifying the last line of the above code snippet (the trackPageView command) or adding additional code to capture more interactions. However, you should not change the code that loads the collect.js library or initializes the `_etmc.push()` command queue function. Refer to the [official docs](https://help.salesforce.com/articleView?id=mc_ctc_collect_code.htm&type=5) for details on available options.

### Asynchronous Collect Code with preloading

The above Collect Code snippet will ensure things work across all browsers but it has the disadvantage of not allowing modern browsers to preload the script.

The alternative async tag below adds support for preloading, which will provide a small performance boost on modern browsers, but can degrade to synchronous loading and execution on IE 9 and older mobile browsers that do not recognize the async script attribute. Only use this tag configuration if your visitors primarily use modern browsers to access your site.

To initialize the tracking, do not forget to replace the two occurences of "INSERT_MID" with your actual BU's Member ID.

```html
<!-- Einstein Collect Code -->
<script>
window._etmc=window._etmc||[];

_etmc.push(['setOrgId', 'INSERT_MID']);
_etmc.push(['trackPageView']);
</script>
<script async src='//INSERT_MID.collect.igodigital.com/collect.js'></script>
<!-- End Einstein Collect Code -->
```

Keep the above in the `<head>` section of your code.

Download: [Async preload sample code](collect-code/collect-code.async-preload.html)
