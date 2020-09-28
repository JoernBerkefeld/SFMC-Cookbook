# Einstein Recommendations <!-- omit in toc -->

This page aims to make using Einstein recommendations a little easier by adding a few explanations around how to use it in a modern setup.

- [Collect Code](#collect-code)
	- [Note to developers](#note-to-developers)
	- [Initialize the library](#initialize-the-library)
		- [Collect Code on page load](#collect-code-on-page-load)
		- [Asynchronous Collect Code](#asynchronous-collect-code)
		- [Asynchronous Collect Code with preloading](#asynchronous-collect-code-with-preloading)
	- [Tracking and other Collect Code features](#tracking-and-other-collect-code-features)
	- [Debug your tracking solution](#debug-your-tracking-solution)
	- [Track Page Views: trackPageView](#track-page-views-trackpageview)
	- [Disable tracking](#disable-tracking)
- [Update Catalog](#update-catalog)
	- [Via Collect Code](#via-collect-code)
	- [Update Catalog via API](#update-catalog-via-api)
- [Einstein Email Recommendations](#einstein-email-recommendations)
- [Einstein Web Recommendations](#einstein-web-recommendations)

## Collect Code

> Official documentation: [help.salesforce.com](https://help.salesforce.com/articleView?id=mc_ctc_collect_code.htm&type=5)

The default sample codes all assume a page load is happening whenever something needs to be tracked. However, the `collect.js` library is actually capable of being loaded asynchronously and can be used in a single-page-application (SPA) environment as well, similarly to [Google's analytics.js](https://developers.google.com/analytics/devguides/collection/analyticsjs).

### Note to developers

While the offical sample code snippets might lead you to believe that you are looking at an **Array**, that is in fact only half true. The variable `_etmc` gets rewritten by `collect.js` to become an object that happens to also expose a method named `push()`. Their base snippet in fact assumes that `collect.js` has been loaded synchronously before you execute your first call to `push()`. Essentially, the first Array-element is the name of an internal method of collect.js that gets executed.

For more details, have a look at the [collect.js](collect-code/default/collect.js) library itself.

### Initialize the library

#### Collect Code on page load

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


// then execute tracking, e.g.:
// _etmc.push(['trackPageView']);
</script>
```

Download: [Default sample code](collect-code/default/collect-code.defaut.html)

#### Asynchronous Collect Code

The Collect Code tag should be added near the top of the `<head>` tag and before any other script or CSS tags.

To initialize the tracking, do not forget to replace the two occurences of "INSERT_MID" with your actual BU's Member ID.

```html
<!-- Einstein Collect Code -->
<script>
(function(e,t,c,n,o,s,a){e[o]=e[o]||[],s=t.createElement(c),a=t.getElementsByTagName(c)[0],s.async=1,s.src=n,a.parentNode.insertBefore(s,a)})(window,document,'script','//INSERT_MID.collect.igodigital.com/collect.js','_etmc');

// always run this line once, followed by what you actually want to track; can be run programmatically in single-page-applications
_etmc.push(['setOrgId', 'INSERT_MID']);


// then execute tracking, e.g.:
// _etmc.push(['trackPageView']);
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

#### Asynchronous Collect Code with preloading

The above Collect Code snippet will ensure things work across all browsers but it has the disadvantage of not allowing modern browsers to preload the script.

The alternative async tag below adds support for preloading, which will provide a small performance boost on modern browsers, but can degrade to synchronous loading and execution on IE 9 and older mobile browsers that do not recognize the async script attribute. Only use this tag configuration if your visitors primarily use modern browsers to access your site.

To initialize the tracking, do not forget to replace the two occurences of "INSERT_MID" with your actual BU's Member ID.

```html
<!-- Einstein Collect Code -->
<script>
window._etmc=window._etmc||[];

_etmc.push(['setOrgId', 'INSERT_MID']);


// then execute tracking, e.g.:
// _etmc.push(['trackPageView']);
</script>
<script async src='//INSERT_MID.collect.igodigital.com/collect.js'></script>
<!-- End Einstein Collect Code -->
```

Keep the above in the `<head>` section of your code.

Download: [Async preload sample code](collect-code/collect-code.async-preload.html)

### Tracking and other Collect Code features

There are a lot of options available from the Collect Code library, available via the `_etmc.push()` method:

| Method |Description |
| -- | -- |
| [doNotTrack](https://help.salesforce.com/articleView?id=mc_ctc_do_not_track.htm&type=5) | Deactivate tracking for current user; needs to be executed on page load before other push-calls. |
| setFirstParty | ability to send tracking data to a server other than the default. Use together with one of the `track...` methods |
| setInsecure | Allows to track data on a non-secure server if the current website was not opened securely either; use together with one of the `track...` methods |
| setOrgId | set your BUs MID; use together with one of the `track...` methods |
| [setUserInfo](https://help.salesforce.com/articleView?id=mc_ctc_set_user_info.htm&type=5)        | allows to send in an object with user data `{email:'', custom:'abc'}`; use together with one of the `track...` methods |
| [trackCart](https://help.salesforce.com/articleView?id=mc_ctc_track_cart.htm&type=5) | Log items added or removed from a contact's cart |
| [trackConversion](https://help.salesforce.com/articleView?id=mc_ctc_track_conversion.htm&type=5) | Log details about a contact’s purchase |
| trackEvent | **???** |
| [trackPageView](#track-page-views-trackpageview) * | Log [content/product page](https://help.salesforce.com/articleView?id=mc_ctc_track_page_view.htm&type=5) views, [in-site search terms](https://help.salesforce.com/articleView?id=mc_ctc_track_in_site_search.htm&type=5) and [category views](https://help.salesforce.com/articleView?id=mc_ctc_track_category_view.htm&type=5). More info below |
| [trackRating](https://help.salesforce.com/articleView?id=mc_ctc_track_rating.htm&type=5) | Log a user's rating for an item on your website. |
| trackWishlist | **???** |
| [updateItem](#via-collect-code) * | This allows you to [update your product catalog](https://help.salesforce.com/articleView?id=mc_ctc_streaming_updates.htm&type=5). More info below. |

### Debug your tracking solution

Use the following to activate `console.log` outputs whenever data is send to the server.

```javascript
// enable debug output
_etmc.debug = true;
```

Alternatively, look for loaded images in the Network tab of your browser. The calls go to "files" (endpoints) with the name of your method calls, though rewritten to [Snake Case](https://en.wikipedia.org/wiki/Snake_case).

### Track Page Views: trackPageView

The first element you pass in basically represents a method name which takes multiple variables. `trackPageView` accepts the following parameters alone or combined:

To track the view of a content or product, use this code:

| Key          | Value                   |
| ------------ | ----------------------- |
| `'item'`     | _Product code_ (String) |
| `'search'`   | _Search term_ (String)  |
| `'category'` | _Catgeory_ (String)     |

The most simple versions use **one** of the following lines

```javascript
// product page viewed
_etmc.push(['trackPageView', { 'item' : 'INSERT_PRODUCT_CODE' }]);

// category viewed
_etmc.push(['trackPageView', { 'category' : 'INSERT_CATEGORY' }]);

// search executed
_etmc.push(['trackPageView', { 'search' : 'INSERT_SEARCH_TERM' }]);
```

But of course these can also be combined: If the user came to the page using your search you can optionally use the following extended snippet:

```javascript
_etmc.push(['trackPageView', { 'item' : 'INSERT_PRODUCT_CODE','search' : 'INSERT_SEARCH_TERM' }]);
```

### Disable tracking

Contrary to the docs, just a single line is needed, which then literally deactivates `_etmc.push()`.

```javascript
// disable tracking for current user
_etmc.push(["doNotTrack"]);
```

## Update Catalog

There are various ways of updating the catalog in Einstein that come with their own advantages and disadvantages.

### Via Collect Code

> Source: [help.salesforce.com/articleView?id=mc_ctc_streaming_updates.htm](https://help.salesforce.com/articleView?id=mc_ctc_streaming_updates.htm&type=5)

It is actually possible to send in updates to your catalog via the collect code. This approach runs without authentication and is therefore up for being hacked. Treat it with caution. With that said, you cannot deactivate it AFAIK.

**Update a single item:**

```javascript
_etmc.push(["updateItem",
    {
    "item": "INSERT_ITEM",
    "unique_id": "INSERT_UNIQUE_ITEM_ID",
    "name": "INSERT_ITEM_NAME_OR_TITLE",
    "url": "INSERT_ITEM_URL",
    "item_type": "INSERT_ITEM_TYPE",
    "INSERT_ATTRIBUTE_NAME": "INSERT_ATTRIBUTE_VALUE"
    }
]);
```

**Update multiple items:**

```javascript
_etmc.push(["updateItem", [
    {
        "item": "INSERT_ITEM",
        "unique_id": "INSERT_UNIQUE_ITEM_ID",
        "name": "INSERT_ITEM_NAME_OR_TITLE",
        "url": "INSERT_ITEM_URL",
        "item_type": "INSERT_ITEM_TYPE",
        "INSERT_ATTRIBUTE_NAME": "INSERT_ATTRIBUTE_VALUE"
    },
    {
        "item": "INSERT_ITEM",
        "unique_id": "INSERT_UNIQUE_ITEM_ID",
        "name": "INSERT_ITEM_NAME_OR_TITLE",
        "url": "INSERT_ITEM_URL",
        "item_type": "INSERT_ITEM_TYPE",
        "INSERT_ATTRIBUTE_NAME": "INSERT_ATTRIBUTE_VALUE"
    }
]]);
```

### Update Catalog via API

> Source: [help.salesforce.com/articleView?id=mc_ctc_streaming_updates.htm](https://help.salesforce.com/articleView?id=mc_ctc_streaming_updates.htm&type=5)

This is in theory possible, however so far I haven't gotten it to work.

## Einstein Email Recommendations

> Official docs: [help.salesforce.com/articleView?id=mc_pb_einstein_email_recommendations.htm](https://help.salesforce.com/articleView?id=mc_pb_einstein_email_recommendations.htm&type=5)

## Einstein Web Recommendations

> Official docs: [help.salesforce.com/articleView?id=mc_pb_einstein_web_recommendation.htm](https://help.salesforce.com/articleView?id=mc_pb_einstein_web_recommendation.htm&type=5)
