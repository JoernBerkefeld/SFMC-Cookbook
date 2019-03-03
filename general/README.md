# General Guide on SFMC developing

- [General Guide on SFMC developing](#general-guide-on-sfmc-developing)
	- [Development Setup](#development-setup)
		- [Where to code](#where-to-code)
		- [Use Live Linting & Auto-Formatting](#use-live-linting--auto-formatting)
		- [File structure](#file-structure)
		- [How to debug quickly](#how-to-debug-quickly)

## Development Setup

### Where to code

By all means, avoid making changes directly in SFMC's GUI. Best practice is to make all your changes in your local IDE instead (I recommend the free [Visual Studio Code](https://code.visualstudio.com/)).

_Why?_ Whenever you save a content block, there is no going back. No CTRL+Z, no undo last change - nothing. Your previous code is imply gone.

_Why?_ If others open your CloudPage and accidentally make changes you (a) will never find out who did it and (b) have no way of recovering the previous version.

_Why?_ The GUI does not offer proper code highlighting (even if you wrap your code in `<script>...</script>`)

_Why?_ The GUI does not offer any linting / error correction. In your IDE you can utilize [ESLint](https://eslint.org/) for your SSJS and JS code, as well as [Prettier](https://prettier.io/) for your CSS

### Use Live Linting & Auto-Formatting

Now, this unfortunately is NOT yet available for AMPscript but one day in the future... well either AMPscript is gone or somebody will have written a plugin. Aside from that, for HTML, CSS, JavaScript and whatever preprocessor you might want to use in your project, there are powerful tools available that can auto-fix minor code issues and enforce code formatting rules.

_Why?_ Linter's will tell you about potential code problems before they are published to PROD, making your life a hell lot easier.

_Why?_ Having to adapt to multiple styles in one project makes the code harder to read and more difficult to debug.

_Why?_ Your client will likely ask you to adhere to certain standards. Adapting your default configuration to these client standards usually reduces time spend on code reviews to the bare minimum.

_Why?_ Junior developers can learn from how their code is changed by the auto-formatters and

_Why?_ Still not convinced? The guys over at Prettier wrote an actual [essay](https://prettier.io/docs/en/why-prettier.html) on this.

### File structure

Any file should only contain markup language (HTML) or front-end code (CSS or JS) or AMPscript code or SSJS. You should avoid mixing this and mimic what the structure you saved locally.
To still be able to easily recognize which content block contains what, add a little AMPscript at the beginning:

```java
%%[ /* myAMPscript.amp */ ]%%

%%[
// insert AMPscript code here
]%%
```

or

```java
%%[ /* mySSJS.ssjs */ ]%%

<script runat="server" language="JavaScript">
// insert SSJS code here
</script>
```

### How to debug quickly

Ever tried to fix a bug and got annoyed by having to publish your page or send your email 1000 times per day? Turns out you can load **any** text file that’s freely available on the internet and run it through the server-side engines for both AmpScript and SSJS. The key is a method that’s similar to JavaScript’s `eval()` method together with another method that grabs the text content of a URL and returns it as a String:

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

_Thanks_ to my buddy Christian published a [full tutorial](https://simple-force.com/2018/12/17/marketing-cloud-best-practice-external-editors/) on this a while ago and taught me this.
