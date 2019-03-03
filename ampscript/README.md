# The AMPscript Coding Guide

- [The AMPscript Coding Guide](#the-ampscript-coding-guide)
	- [Hide your code](#hide-your-code)

## Hide your code

When AMPscript is inserted into either an email or a CloudPage it usually clutters up the preview window with code. To avoid this, wrap your code in something that will not render:

```html
<div style="display:none">
%%[
// your code here
]%%
</div>
```

Why not something that does not render out of the box, like `<script>...</script>` or `<style>...</style>`? SFMC will remove all `<script>` tags out of emails without a prompt. Otherwise, they could actually be seen as helpful as they lead to some kind of code highlighting even for AMPscript. However, both might also lead to bad reformatting schemes being applied, potentially breaking your code.

_Why?_ Non-developers will just get distracted by the code

_Why?_ Showing lots of code during a demo in the preview window distracts from the actual email content and might cost you time explaining this again and again.

_Why?_ Not seeing the Code shrinks the content block to its minimum height, making selecting the right block easier as scrolling occurs less often.
