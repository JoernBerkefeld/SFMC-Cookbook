# 1. Frequently Asked Questions

- [1.1. Automations](#11-automations)
  - [1.1.1. How do I test automations? 'Run once' does not show any progress.](#111-how-do-i-test-automations-run-once-does-not-show-any-progress)
- [1.2. CloudPages](#12-cloudpages)
  - [1.2.1. I want to publish my CloudPage but the interface seems like it's stuck?](#121-i-want-to-publish-my-cloudpage-but-the-interface-seems-like-its-stuck)
  - [1.2.2. I changed the name of my CloudPage but it still shows up with a different name.](#122-i-changed-the-name-of-my-cloudpage-but-it-still-shows-up-with-a-different-name)
  - [1.2.3. How do I change the HTML title or SEO attributes of my Cloudpage?](#123-how-do-i-change-the-html-title-or-seo-attributes-of-my-cloudpage)
  - [1.2.4. How can I set a custom URL / path for my CloudPage? It always shows auto-generated URLs instead.](#124-how-can-i-set-a-custom-url--path-for-my-cloudpage-it-always-shows-auto-generated-urls-instead)
- [1.3. Emails](#13-emails)
  - [1.3.1. How do I test if my email code works?](#131-how-do-i-test-if-my-email-code-works)
  - [1.3.2. I want to send a triggered send email with custom data coming from the API call](#132-i-want-to-send-a-triggered-send-email-with-custom-data-coming-from-the-api-call)
- [1.4. Journey Builder](#14-journey-builder)
  - [1.4.1. How can I enable that contacts should not re-enter my Journey?](#141-how-can-i-enable-that-contacts-should-not-re-enter-my-journey)
- [1.5. Setup (Admin) menu](#15-setup-admin-menu)
  - [1.5.1. How can I add new users? – I have access, but I get error messages when I want to save.](#151-how-can-i-add-new-users--i-have-access-but-i-get-error-messages-when-i-want-to-save)

## 1.1. Automations

### 1.1.1. How do I test automations? 'Run once' does not show any progress.

There is a bug in SFMC's UI (at time of writing, 2019-07-14) that will not show you any progress when using the 'Run once'-button - you are forced to wait and reload the page again and again. Instead, schedule the import for 1 or 2 minutes later, have it run and then disable the schedule again. Doing it this way, you get to see a proper progress bar that updates automatically (might want to try reloading if auto-update seems broken - it won't do any harm).

## 1.2. CloudPages

### 1.2.1. I want to publish my CloudPage but the interface seems like it's stuck?

It's not a feature, it's a bug - on two ends in this case: Content Builder's publish-dialogue unfortunately has no proper error output. It will literally have you die waiting for it to finish if any code errors were noticed. But behold, there is an easy workaround: If you click on "Publish" in Google's Chrome browser, you can hit the F12 key to open "Chrome Dev Tools". This should show you a multitude of tools but most importantly the first tab thats always active is the "Console". Make it a habit to open this before you hit the Publish-button, then wait if among the many (usually yellow) messages the last 2-3 all the sudden are in red and mention something about a **server error**. That ususally takes only a few seconds to appear and you know you can click on the "Cancel"-button and go find the bug - do **NOT** click on the "Publish"-button in this dialogue or you will break your live version!

### 1.2.2. I changed the name of my CloudPage but it still shows up with a different name.

This is a bug in SFMC's logic but there is a workaround: You have to change the name in two different menues to have the new name displayed everywhere. First, click on "Edit Content"-button which takes you to the interface that allows changing the code. Update the name in the upper right corner. Second, going back to the collection page, instead of the "Edit Content"-button, click on the little burger-icon (button with 3 lines) on the CloudPage in question. That takes you to a settings page. Here, you can update all kinds of things, including the title that is shown in the Collection page.

### 1.2.3. How do I change the HTML title or SEO attributes of my Cloudpage?

Go to the collection that holds your page. Click on the burger-icon displayed when you hover your CloudPage. Here you have all kinds of advanced settings, including the title, keywords and search engine visibility.

### 1.2.4. How can I set a custom URL / path for my CloudPage? It always shows auto-generated URLs instead.

This feature is only enabled after you linked a custom domain to the BU that you are working on. Please contact SFMC support to find out more details on the current process.

## 1.3. Emails

### 1.3.1. How do I test if my email code works?

The fastest approach is to write your code in a CloudPage and then copy it to an email / content block later. However, once you need to include attributes from an associated Data Extension that will not help you anymore. Also, there are some methods that work in CloudPages but not in Emails, hence don't be shocked if your code stops working once you do copy it over. In that case, you need to proceed to the last (third) tab which would let you send out test messages. When that page loaded it will show you error messages if your code has problems. Do also select a test user from the data extension you are planning to use to make sure the preview looks right - you could still have logic problems in your solution that are not syntax related.

### 1.3.2. I want to send a triggered send email with custom data coming from the API call

This one is a bit tricky, however possible in both SSJS and AmpScript:

```json
// {{hostEndpoint}}/messaging/v1/messageDefinitionSends/key:{{TriggeredSend}}/sendBatch
[
    {
        "To": {
            "Address": "joern.berkefeld@some-mail.com",
            "SubscriberKey": "123abc123abc123abc123abc123abc",

            "ContactAttributes": {
                "SubscriberAttributes": {
                    "myCustomAttribute1": "some value",
                    "myCustomAttribute2": 4,
                    "myCustomAttribute3": false,
                    "myCustomAttribute4": "{\"complex-objects\":\"need to be stringified\"}",
                    "myCustomAttribute5": "<some><xml>here</xml><xml>or here</xml></some>"
                }
            }
        }
    }
]
```

AmpScript solution:

```c++
# ampscript
%%[
// get attributes passed in during the API call
SET @payload1 = AttributeValue("myCustomAttribute1")
/* ... */
SET @payload5 = AttributeValue("myCustomAttribute5")
// note that payload5 is still just a string at this point. You will now need to decode it
]%%
```

Server-Side JavaScript solution:

```javascript
function jsonParse(str) {
    return Platform.Function.ParseJSON(str + '');
}
// no AMPscript needed, just use the same function without the leading "@"
var payload1 = Variable.GetValue('myCustomAttribute1');
/* ... */
var payload4 = Variable.GetValue('myCustomAttribute4');
// now convert payload4's string into an actual JSON
// the "+ ''" converts payload4 into a string, making sure we dont get error 500 if it wasn't already
payload4 = Platform.Function.ParseJSON(payload4 + '');
```

_Note:_ It is recommended by SFMC (at least the TC that I talked to) as well as by myself to only use ONE parameter and encode all your data inside of it as either XML (for AmpScript) or JSON (for SSJS). That way, you are more flexible when the system calling you API wants to send in more data.

## 1.4. Journey Builder

### 1.4.1. How can I enable that contacts should not re-enter my Journey?

This can only be done when creating your Journey or a new version of it. If you have already published your Journey, create a new version, change the settings according to your will and then set this new version to active.

## 1.5. Setup (Admin) menu

### 1.5.1. How can I add new users? – I have access, but I get error messages when I want to save.

Please make sure you have access to the global/parent BU and have selected that when you try adding users. Marketing Cloud’s UI is misleading and will allow you to open all the proper forms but block you in the last moment after you clicked on the “Save”-button.
SMS
