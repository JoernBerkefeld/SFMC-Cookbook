# Einstein Recommendations <!-- omit in toc -->

This page aims to make using Einstein recommendations a little easier by adding a few explanations around how to use it in a modern setup.

- [Collect Code](#collect-code)
  - [Note to developers](#note-to-developers)
  - [Initialize the library](#initialize-the-library)
    - [Collect Code on page load](#collect-code-on-page-load)
    - [Asynchronous Collect Code](#asynchronous-collect-code)
    - [Asynchronous Collect Code with preloading](#asynchronous-collect-code-with-preloading)
  - [Debug your tracking solution](#debug-your-tracking-solution)
  - [Tracking and other Collect Code features](#tracking-and-other-collect-code-features)
    - [Disable tracking](#disable-tracking)
    - [Identify Business Unit for Tracking](#identify-business-unit-for-tracking)
    - [Identify current user](#identify-current-user)
      - [Attribute Affinity](#attribute-affinity)
    - [Track Page Views: trackPageView](#track-page-views-trackpageview)
    - [Track Items in Cart: trackCart](#track-items-in-cart-trackcart)
    - [Track Purchases / Conversions: trackConversion](#track-purchases--conversions-trackconversion)
    - [Track User Wishlist: trackWishList](#track-user-wishlist-trackwishlist)
- [Update Catalog](#update-catalog)
  - [Via Collect Code](#via-collect-code)
  - [Update Catalog via API](#update-catalog-via-api)
- [Predictive Intelligence (PI) Data Extensions](#predictive-intelligence-pi-data-extensions)
- [Einstein Email Recommendations](#einstein-email-recommendations)
- [Einstein Web Recommendations](#einstein-web-recommendations)
  - [Embedding Web Recommendations](#embedding-web-recommendations)
    - [How the recommender knows who the current user is](#how-the-recommender-knows-who-the-current-user-is)
    - [Enhancing recommendation results](#enhancing-recommendation-results)
    - [Embed via JSON](#embed-via-json)
      - [JSON Example responses](#json-example-responses)
    - [Embed via JavaScript ("HTML")](#embed-via-javascript-html)
      - [JavaScript/HTML example code](#javascripthtml-example-code)

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

Download: [Default sample code](https://github.com/JoernBerkefeld/SFMC-Cookbook/blob/master/einstein/recommendation/collect-code/default/collect-code.defaut.html)

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

Download: [Async sample code](https://github.com/JoernBerkefeld/SFMC-Cookbook/blob/master/einstein/recommendation/collect-code/collect-code.async.html)

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

Download: [Async preload sample code](https://github.com/JoernBerkefeld/SFMC-Cookbook/blob/master/einstein/recommendation/collect-code/collect-code.async-preload.html)

### Debug your tracking solution

Use the following to activate `console.log` outputs whenever data is send to the server.

```javascript
// enable debug output
_etmc.debug = true;
```

Alternatively, look for loaded images in the Network tab of your browser. The calls go to "files" (endpoints) with the name of your method calls, though rewritten to [Snake Case](https://en.wikipedia.org/wiki/Snake_case).

### Tracking and other Collect Code features

There are a lot of options available from the Collect Code library, available via the `_etmc.push()` method. Only options starting on "track" and "update" actually result in a callout to the server. The "set" and "doNotTrack" methods are mere settings that need to be executed before those.

| Method |Description |
| -- | -- |
| [doNotTrack](#disable-tracking) * | Deactivate tracking on the current page. More info below. |
| setFirstParty | Allows you to send tracking data to a server other than the default, e.g. to proxy the data through your own server. Use together with one of the `track...` methods |
| setInsecure | Use together with `setFirstParty` to track data via a non-secure proxy server. Only works if the current website was not opened securely either; use together with one of the `track...` methods |
| [setOrgId](#identify-business-unit-for-tracking) * | set your BUs MID; use together with one of the `track...` methods |
| [setUserInfo](#identify-current-user) * | allows to send in an object with user data `{email:'', custom:'abc'}`; use together with one of the `track...` methods |
| [trackCart](#track-items-in-cart-trackcart) * | Log items added or removed from a contact's cart |
| [trackConversion](https://help.salesforce.com/articleView?id=mc_ctc_track_conversion.htm&type=5) | Log details about a contact’s purchase |
| trackEvent | _Undocumented feature:_ Allows you to track custom events |
| [trackPageView](#track-page-views-trackpageview) * | Log [content/product page](https://help.salesforce.com/articleView?id=mc_ctc_track_page_view.htm&type=5) views, [in-site search terms](https://help.salesforce.com/articleView?id=mc_ctc_track_in_site_search.htm&type=5) and [category views](https://help.salesforce.com/articleView?id=mc_ctc_track_category_view.htm&type=5). More info below |
| [trackRating](https://help.salesforce.com/articleView?id=mc_ctc_track_rating.htm&type=5) | Log a user's rating for an item on your website. |
| [trackWishlist](#track-user-wishlist-trackwishlist) * | _Undocumented feature:_ Allows you to track multiple "shopping carts"-like lists in which users track future wishes |
| [updateItem](#via-collect-code) * | This allows you to [update your product catalog](https://help.salesforce.com/articleView?id=mc_ctc_streaming_updates.htm&type=5). More info below. |

#### Disable tracking

Contrary to the [official docs](https://help.salesforce.com/articleView?id=mc_ctc_do_not_track.htm&type=5), just a single line is needed, which then literally deactivates `_etmc.push()` and therefore any other tracking calls that are issued afterwards on the current page. This should be executed on page load before other push-calls.

```javascript
// disable tracking for current page
_etmc.push(['doNotTrack']);
```

#### Identify Business Unit for Tracking

This is a required configuration step before tracking anything which allows the collect code to send the tracking data to the right business unit.

```javascript
_etmc.push(['setOrgId','INSERT_MID']);
```

#### Identify current user

Contrary to what the [official docs](https://help.salesforce.com/articleView?id=mc_ctc_set_user_info.htm) state, the only line required to define the user is this:

```javascript
_etmc.push(['setUserInfo', {'email': 'INSERT_EMAIL_OR_UNIQUE_ID'}]);
```

It seems that if one wants to use [Predictive Intelligence integration](https://help.salesforce.com/articleView?id=mc_pb_integration_with_contact_builder.htm&type=5) completely, one has to use the actual email address for `INSERT_EMAIL_OR_UNIQUE_ID`. This assumption is supported by the automatically created attribute set that links the PI_* Data Extensions to a Contact using the Email, rather than the subscriber key.

On the other hand, if you only care about showing Einstein recommendations, you simply have to ensure that you use the same string when you retrieve web/email recommendations that you previously used for tracking via collect code.

> **Note:** It is so far not completely clear to me what effect it has if you don't use the readable email here but instead follow Salesforce's optional recommendation to hash/encrypt the email address. Will update this section once I tested more.

##### Attribute Affinity

This should theoretically boost catalog items that carry the same attribute (detail-field and value) defined as the current user. [Quote](https://help.salesforce.com/articleView?id=mc_ctc_set_contact_attribute_affinity.htm&type=5):

> Match a contact attribute to a tagged catalog field to increase the subscriber's affinity for the value of that contact attribute. The amount of increase is less than what results from a purchase but more than the increase from a view.

```javascript
_etmc.push(['setUserInfo', {
    'email': 'INSERT_EMAIL_OR_UNIQUE_ID',
    'details': {
        'gender': 'female',
        'otherCustomAttribute', 'myValue'
    }
}]);
```

#### Track Page Views: trackPageView

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

#### Track Items in Cart: trackCart

This should be run each time a product is added or removed from the cart, the quantity is changed or when the purchase is finalized.

```javascript
_etmc.push(['trackCart', {
    'cart': [
        {
            'item': 'INSERT_ITEM',
            'quantity':  'INSERT_QUANTITY',
            'price': 'INSERT_PRICE',
            'unique_id': 'INSERT_UNIQUE_ID'
        },
        {
            'item': 'INSERT_ITEM',
            'quantity':  'INSERT_QUANTITY' ,
            'price': 'INSERT_PRICE' ,
            'unique_id': 'INSERT_UNIQUE_ID'
        }
    ]
}]);
```

**Definitions:**
| Key | Definition |
| -- | -- |
| `item`      | Matches the field mapped to **ProductCode** in the catalog. |
| `quantity`  | The number of items added for the particular SKU. |
| `price`     | The price at the time of adding an item to the cart. |
| `unique_id` | Matches the field mapped to the **SKUId** in the catalog. When these items match it ensures the exact record in the catalog, including all specific attributes like color and size, is tied to the cart rather than just the ProductCode |

**Important:** Always include the entire cart in this call because it will overwrite whatever was stored before. Therefore, in order to remove one row, simply pass in all other rows that were not deleted.

The [official docs](https://help.salesforce.com/articleView?id=mc_ctc_track_cart.htm&type=5) state that one should use the following to remove all cart line items at once:

```javascript
_etmc.push(['trackCart', { 'clear_cart': true } ]);
```

It is likely that sending in an empty Array (`{ 'cart': [] }`) also works but this is untested at this point.

#### Track Purchases / Conversions: trackConversion

```javascript
_etmc.push(['trackConversion', {
    'cart': [
        {
            'item': 'INSERT_ITEM',
            'quantity':  'INSERT_QUANTITY',
            'price': 'INSERT_PRICE',
            'unique_id': 'INSERT_UNIQUE_ID'
        },
        {
            'item': 'INSERT_ITEM',
            'quantity':  'INSERT_QUANTITY',
            'price': 'INSERT_PRICE',
            'unique_id': 'INSERT_UNIQUE_ID'
        }
    ],
   // OPTIONAL PARAMETERS
   'order_number': 'INSERT_ORDER_NUMBER',
   'discount': 'INSERT_DISCOUNT',
   'shipping': 'INSERT_SHIPPING',
   'details': {
       'AttributeName': 'Value'
    }
   // END OPTIONAL PARAMETERS
}]);
```

| Key          | Description                                                                           |
| ------------ | ------------------------------------------------------------------------------------- |
| cart         | same as for `trackCart`; [see above](#track-items-in-cart-trackcart) for more details |
| order_number | _optional:_ String                                                                    |
| discount     | _optional:_ Decimal, in local currency, not percental value                           |
| shipping     | _optional:_ Decimal, in local currency, not percental value                           |
| details      | _optional:_ ???                                                                       |

**Note:**
If `shipping` or `discount` is set, the line item `price` values will actually be calculated keeping these in mind rather than storing discount and shipping separately:

```javascript
// example call
_etmc.push(['trackConversion', {
    'cart': [
        {
            'item': '123',
            'quantity': '2',
            'price': '10.00',
            'unique_id': '123'
        },
        {
            'item': '234',
            'quantity': '1',
            'price': '5.00',
            'unique_id': '234'
        }
    ],
    // OPTIONAL PARAMETERS
    'order_number': '123456',
    'discount': '2.00',
    'shipping': '5.00'
    // END OPTIONAL PARAMETERS
}]);
```

results in...

```javascript
// (pseudo-code)
var sumProductPrices = FOREACH(quantity*price) = 2*10 + 1*5 = 25
var finalItemPrice[0] = price + ((shipping - discount) * (quantity*price/sumProductPrices) / quantity = 10 + ((5-2)*(2*10/25)/2) = 11.20
```

#### Track User Wishlist: trackWishList

Allows you to store contact wishlists for items on your website.

```javascript
_etmc.push(['trackWishlist', {
    'items' : ['INSERT_ITEM_1', 'INSERT_ITEM_2', 'INSERT_ITEM_3'],
    'skus' : ['INSERT_UNIQUE_ID_1', 'INSERT_UNIQUE_ID_2', 'INSERT_UNIQUE_ID_3']
}]);
```

| Key         | Definition                                                  |
| ----------- | ----------------------------------------------------------- |
| `item`      | Matches the field mapped to **ProductCode** in the catalog. |
| `unique_id` | Matches the field mapped to the **SKUId** in the catalog.   |

According to the offical docs, handing in skus is optional but I haven't tested that. Make sure the 2 arrays have the same length as the first item in one list is mapped to the first in the other; the second to the second; and so on and so forth.

The contact’s wishlist data is replaced by each subsequent trackWishlist call. Therefore, make sure to always include the entire _current_ list and remove entries from the call that the user deleted from it.

## Update Catalog

There are various ways of updating the catalog in Einstein that come with their own advantages and disadvantages.

### Via Collect Code

> Source: [help.salesforce.com/articleView?id=mc_ctc_streaming_updates.htm](https://help.salesforce.com/articleView?id=mc_ctc_streaming_updates.htm&type=5)

It is actually possible to send in updates to your catalog via the collect code. This approach runs without authentication and is therefore up for being hacked. Treat it with caution. With that said, you cannot deactivate it AFAIK.

**Update a single item:**

```javascript
_etmc.push(['updateItem',
    {
    'item': 'INSERT_ITEM',
    'unique_id': 'INSERT_UNIQUE_ITEM_ID',
    'name': 'INSERT_ITEM_NAME_OR_TITLE',
    'url': 'INSERT_ITEM_URL',
    'item_type': 'INSERT_ITEM_TYPE',
    'INSERT_ATTRIBUTE_NAME': 'INSERT_ATTRIBUTE_VALUE'
    }
]);
```

**Update multiple items:**

```javascript
_etmc.push(['updateItem', [
    {
        'item': 'INSERT_ITEM',
        'unique_id': 'INSERT_UNIQUE_ITEM_ID',
        'name': 'INSERT_ITEM_NAME_OR_TITLE',
        'url': 'INSERT_ITEM_URL',
        'item_type': 'INSERT_ITEM_TYPE',
        'INSERT_ATTRIBUTE_NAME': 'INSERT_ATTRIBUTE_VALUE'
    },
    {
        'item': 'INSERT_ITEM',
        'unique_id': 'INSERT_UNIQUE_ITEM_ID',
        'name': 'INSERT_ITEM_NAME_OR_TITLE',
        'url': 'INSERT_ITEM_URL',
        'item_type': 'INSERT_ITEM_TYPE',
        'INSERT_ATTRIBUTE_NAME': 'INSERT_ATTRIBUTE_VALUE'
    }
]]);
```

### Update Catalog via API

> Source: [help.salesforce.com/articleView?id=mc_ctc_streaming_updates.htm](https://help.salesforce.com/articleView?id=mc_ctc_streaming_updates.htm&type=5)

This is in theory possible, however so far I haven't gotten it to work.

## Predictive Intelligence (PI) Data Extensions

`PI_ABANDONED_CART_EVENT` and `PI_ABANDONED_CART_ITEMS` will never have any data, according to a [Partner webinar from Salesforce](https://share.vidyard.com/watch/ufsSo83N4VaCSpZzxhS7Eo). They were "part of an earlier attempt" of the solution but are no longer in use - but nonetheless still created together with your BUs.

## Einstein Email Recommendations

> Official docs: [help.salesforce.com/articleView?id=mc_pb_einstein_email_recommendations.htm](https://help.salesforce.com/articleView?id=mc_pb_einstein_email_recommendations.htm&type=5)

**TODO:** Add more details :-)

## Einstein Web Recommendations

> Official docs: [help.salesforce.com/articleView?id=mc_pb_einstein_web_recommendation.htm](https://help.salesforce.com/articleView?id=mc_pb_einstein_web_recommendation.htm&type=5)

To get recommendations, you have to create "pages" that should mirror the views/pages of your website. You can define per page what exactly shall be recommended and also choose between JSON and JavaScript format.

### Embedding Web Recommendations

The JSON has to be retrieved via XHR callout and then parsed by your own code to actually create visible output in your HTML. The JavaScript approach requires you to load the JS file like any other JavaScript resource but also include  pre-defined HTML code, provided to you by the "Get Code" tab.

Please note that you select the output format when creating a "page" in Einstein on the "ouput" tab. This will define what code is presented on the "Get Code" tab

![selecting output format](img/select-output-format.jpg)

You should define what field values you want in your recommendation via the "Output" tab. Here you can add, remove and order the fields that will be available in your recommendation. Ordering has no impact if you choose to embed via JSON.

#### How the recommender knows who the current user is

The current user is actually shared via cookie set by the Collect Code (collect.js). This might lead to other challenges given the browser initiatives to block third-party cookies (see CORS) and explains what `setFirstParty` is for. You might have to implement a proxy logic to accomodate this.

If no cookie was set, generic recommendations are displayed.

#### Enhancing recommendation results

You can append parameters to the recommend.json / recommend.js as URL parameters to get more focused results. Please make sure you URL-Encode the values!

| URL Parameter | value           |
| ------------- | --------------- |
| item          | _SKU_           |
| cart          | _SKU_           |
| category      | _CATEGORY_NAME_ |
| search        | _SEARCH_TERM_   |

**Example for JSON:**

```javascript
GET https://INSERT_MID.recs.igodigital.com/a/v2/INSERT_MID/INSERT_PAGE_NAME/recommend.json?category=My%20Shoes&search=red%20female
```

**Example for HTML/JavaScript:**

```html
<!-- Copy this code right before the closing </body> of your INSERT_PAGE_NAME page-->
<script src="https://INSERT_MID.recs.igodigital.com/a/v2/INSERT_MID/INSERT_PAGE_NAME/recommend.js?category=My%20Shoes&search=red%20female"></script>
```

#### Embed via JSON

This requires you to do all the styling and processing yourself but also comes with greatest amount of flexibility. Call the below URL to get the JSON for your BU-Page combo:

```javascript
GET https://INSERT_MID.recs.igodigital.com/a/v2/INSERT_MID/INSERT_PAGE_NAME/recommend.json
```

The complete URL is provided on the "Get Code" tab, however that page is misleading in other ways:

**Important:** The Web Recommendations' "Get Code" tab was apparently only written with the "html" / JavaScript embed code in mind and falsely asks you to load the JSON via script-tag. It then continues to also ask you to include certain HTML. You need to ignore that and simply copy the url out of that snippet instead!

![Bad embed code for JSON approach](img/bad-json-embed-snippet.jpg)

##### JSON Example responses

The following assumes only one recommendation area was defined and the default name "igdrec_1" was kept. Furthermore, the list of included fields was defined as "ProductLink", "ImageLink", "ProductName", "RegularPrice".

As long as recommendations are not ready, the API will return ab `empty:true` attribute per defined recommendation area:

```json
[
    {
        "name": "igdrec_1",
        "empty": true
    }
]
```

The reponse will look something like this once recommendations are actually available:

```json
[
    {
        "name": "igdrec_1",
        "title": "Popular Items Today",
        "priority": 1,
        "items": [
            {
                "link": "Link",
                "image_link": "Image link",
                "name": "Name",
                "regular_price": 112.0
            },
            {
                "link": "Link",
                "image_link": "Image link",
                "name": "Name",
                "regular_price": 112.0
            },
            {
                "link": "Link",
                "image_link": "Image link",
                "name": "Name",
                "regular_price": 112.0
            }
        ]
    }
]
```

#### Embed via JavaScript ("HTML")

You will be asked to load a JavaScript file like this:

```html
<!-- Copy this code right before the closing </body> of your INSERT_PAGE_NAME page-->
<script src='https://INSERT_MID.recs.igodigital.com/a/v2/INSERT_MID/INSERT_PAGE_NAME/recommend.js'></script>
```

As well as position some HTML where you want the final recommendation to be inserted like this:

```html
<!-- Copy this code to where you want to show web recommendations on your INSERT_PAGE_NAME page-->
<div id='igdrec_1'></div>
```

This second part will vary depending on your choices made on the "Build">"Areas" tab.

##### JavaScript/HTML example code

As long as **recommendations are not ready** the code of `recommend.js` will be missing the crucial part that actually fills in the recommendation.

```javascript
// recommend.js if recommendations are not ready yet and only one area named "idgrec_1" was defined for this page

function display_INSERT_PAGE_NAME(zone, id) {
    if (id === 'igdrec_1') {
          zone.innerHTML = '';
    }
}

function addLoadEvent(func) {
    var oldonload = window.onload;
    if (typeof window.onload != 'function') {
        window.onload = func;
    } else {
        window.onload = function() {
            if (oldonload) {
                oldonload();
            }
            func();
        }
    }
}

function callREC() {
    var pageZone = document.getElementById('igdrec_1');
    if ( undefined != pageZone) {
        display_INSERT_PAGE_NAME(pageZone, 'igdrec_1');
    }
}

callREC();
```

**Note:** the method `display_INSERT_PAGE_NAME()` will be named according to your page: If you named the page "MyTest" the method will be named `display_MyTest()`.

If you defined more than one recommendation area, the code will be auto-extended to match that:

```javascript
// recommend.js if recommendations are not ready yet and two areas named "idgrec_1" and "idgrec_2" were defined for this page

function display_INSERT_PAGE_NAME(zone, id) {
    if (id === 'igdrec_1') {
        zone.innerHTML = '';
    }
    if (id === 'igdrec_2') {
        zone.innerHTML = '';
    }
}

function addLoadEvent(func) {
    var oldonload = window.onload;
    if (typeof window.onload != 'function') {
        window.onload = func;
    } else {
        window.onload = function() {
            if (oldonload) {
                oldonload();
            }
            func();
        }
    }
}

function callREC() {
    var pageZone = document.getElementById('igdrec_1');
    if ( undefined != pageZone) {
        display_INSERT_PAGE_NAME(pageZone, 'igdrec_1');
    }
    var pageZone = document.getElementById('igdrec_2');
    if ( undefined != pageZone) {
        display_INSERT_PAGE_NAME(pageZone, 'igdrec_2');
    }
}

callREC();
```

The reponse will look something like this **once recommendations are actually available**:

```javascript
// recommend.js if recommendations are finally ready and only one area named "idgrec_1" was defined for this page

// really curious myself...
```

The HTML that will be created for you will look something like the following:

```html
<div class='igo_boxhead'>
    <h2>Popular Items Today</h2>
</div>
<div class='igo_boxbody'>
    <div class='igo_product'>
        <div class='igo_product_link'>
            <span class='igo_product_link_label'>Link</span>
            <span class='igo_product_link_value'>Link</span>
        </div>
        <div class='igo_product_image_link'>
            <a href=''>
                <img class='igo_product_image' src='Image link'>
            </a>
        </div>
        <div class='igo_product_name'>
            <span class='igo_product_name_label'>Name</span>
            <span class='igo_product_name_value'>Name</span>
        </div>
        <div class='igo_product_regular_price'>
            <span class='igo_product_regular_price_label'>Regular Price</span>
            <span class='igo_product_regular_price_value'>112.0</span>
        </div>
    </div>
    <div class='igo_product'>
        <div class='igo_product_link'>
            <span class='igo_product_link_label'>Link</span>
            <span class='igo_product_link_value'>Link</span>
        </div>
        <div class='igo_product_image_link'>
            <a href=''>
                <img class='igo_product_image' src='Image link'>
            </a>
        </div>
        <div class='igo_product_name'>
            <span class='igo_product_name_label'>Name</span>
            <span class='igo_product_name_value'>Name</span>
        </div>
        <div class='igo_product_regular_price'>
            <span class='igo_product_regular_price_label'>Regular Price</span>
            <span class='igo_product_regular_price_value'>112.0</span>
        </div>
    </div>
    <div class='igo_product'>
        <div class='igo_product_link'>
            <span class='igo_product_link_label'>Link</span>
            <span class='igo_product_link_value'>Link</span>
        </div>
        <div class='igo_product_image_link'>
            <a href=''>
                <img class='igo_product_image' src='Image link'>
            </a>
        </div>
        <div class='igo_product_name'>
            <span class='igo_product_name_label'>Name</span>
            <span class='igo_product_name_value'>Name</span>
        </div>
        <div class='igo_product_regular_price'>
            <span class='igo_product_regular_price_label'>Regular Price</span>
            <span class='igo_product_regular_price_value'>112.0</span>
        </div>
    </div>
</div>
```