# The AMPscript Coding Guide

- [The AMPscript Coding Guide](#the-ampscript-coding-guide)
	- [Hide your code](#hide-your-code)
	- [AMPscript snippets for typical problems](#ampscript-snippets-for-typical-problems)
		- [Use dynamic trackable links](#use-dynamic-trackable-links)
		- [Data Extensions](#data-extensions)
			- [SELECT](#select)
				- [Single column of row, case-insensitive search](#single-column-of-row-case-insensitive-search)
				- [Full row, case-insensitive search](#full-row-case-insensitive-search)
				- [Full row, case-sensitive search](#full-row-case-sensitive-search)
				- [Sorted full row, case-insensitive search](#sorted-full-row-case-insensitive-search)
				- [Sorted full row, case-sensitive search](#sorted-full-row-case-sensitive-search)

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


## AMPscript snippets for typical problems

### Use dynamic trackable links

When creating links that include variables there are multiple approaches but some will lead to tracking being disabled.

```html
<!-- good -->

%%[
SET @myParam = "bar"
SET @url = CONCAT("https://mydomain.com/somePath?foo=", @myParam)
]%%

<a href="%%=RedirectTo(@url)=%%">demo link</a>
```

```html
<!-- bad -->

%%[
SET @myParam = "bar"
SET @url = CONCAT('<a href="', "https://mydomain.com/somePath?foo=", @myParam, '">demo link</a>')
]%%

%%=v(@url)=%%
```

Now, if your link includes parameters, make sure you properly URL-encode the values, otherwise it will fail:

### Data Extensions

#### SELECT

##### Single column of row, case-insensitive search
This method gets one field from the specified data extension; where-filter is case-insensitive.

_Docs [Lookup](https://developer.salesforce.com/docs/atlas.en-us.noversion.mc-programmatic-content.meta/mc-programmatic-content/lookup.htm)_

```c++

Set @dataExtension = 'name of data extension'
Set @returnedField = 'City'
Set @whereCol = 'PostalCode'
Set @whereValue = '12345'

/* get field value from the row according to the filter */
Set @fieldValue = Lookup(@dataExtension, @returnedField, @whereCol, @whereValue)
```

##### Full row, case-insensitive search
This method gets all fields from the specified data extension; where-filter is case-insensitive.

_Docs [LookupRows](https://developer.salesforce.com/docs/atlas.en-us.noversion.mc-programmatic-content.meta/mc-programmatic-content/lookuprows.htm)_
```c++

Set @dataExtension = 'name of data extension'
Set @whereCol = 'some-column'
Set @whereValue = 'something'
Set @rows = LookupRows(@dataExtension, @whereCol, @whereValue)

/* check if @rowCount has at least one row; Row() will fail if it does not */
for @i = 1 to RowCount(@rows) do

	/* select first row */
	Set @row = Row(@rows, @i)

	/* get field values from the row */
	Set @name = Field(@row,'Firstname')
	Set @email = Field(@row,'EmailAddress')

next @i
```

##### Full row, case-sensitive search
This method gets all fields from the specified data extension; where-filter is case-sensitive.

_Docs [LookupRowsCS](https://developer.salesforce.com/docs/atlas.en-us.noversion.mc-programmatic-content.meta/mc-programmatic-content/lookuprowscs.htm)_

```c++

Set @dataExtension = 'name of data extension'
Set @whereCol = 'some-column'
Set @whereValue = 'soMething'
Set @rows = LookupRowsCS(@dataExtension, @whereCol, @whereValue)

/* check if @rowCount has at least one row; Row() will fail if it does not */
for @i = 1 to RowCount(@rows) do

	/* select first row */
	Set @row = Row(@rows, @i)

	/* get field values from the row */
	Set @name = Field(@row,'Firstname')
	Set @email = Field(@row,'EmailAddress')

next @i
```

##### Sorted full row, case-insensitive search
This method gets all fields from the specified data extension; where-filter is case-sensitive.

_Docs [LookupOrderedRows](https://developer.salesforce.com/docs/atlas.en-us.noversion.mc-programmatic-content.meta/mc-programmatic-content/lookuporderedrows.htm)_
```c++

Set @dataExtension = 'name of data extension'
Set @maxReturnedRows = 2000
Set @orderBy = 'MyField ASC' /* 'fieldname ASC' or 'fieldname DESC' */
Set @whereCol = 'some-column'
Set @whereValue = 'soMething'
Set @rows = LookupOrderedRows(@dataExtension, @maxReturnedRows, @orderBy, @whereCol, @whereValue)

/* check if @rowCount has at least one row; Row() will fail if it does not */
for @i = 1 to RowCount(@rows) do

	/* select first row */
	Set @row = Row(@rows, @i)

	/* get field values from the row */
	Set @name = Field(@row,'Firstname')
	Set @email = Field(@row,'EmailAddress')

next @i
```


##### Sorted full row, case-sensitive search
This method gets all fields from the specified data extension; where-filter is case-sensitive.

_Docs [LookupOrderedRowsCS](https://developer.salesforce.com/docs/atlas.en-us.noversion.mc-programmatic-content.meta/mc-programmatic-content/lookuporderedrowscs.htm)_
```c++

Set @dataExtension = 'name of data extension'
Set @maxReturnedRows = 2000
Set @orderBy = 'MyField ASC' /* 'fieldname ASC' or 'fieldname DESC' */
Set @whereCol = 'some-column'
Set @whereValue = 'soMething'
Set @rows = LookupOrderedRowsCS(@dataExtension, @maxReturnedRows, @orderBy, @whereCol, @whereValue)

/* check if @rowCount has at least one row; Row() will fail if it does not */
for @i = 1 to RowCount(@rows) do

	/* select first row */
	Set @row = Row(@rows, @i)

	/* get field values from the row */
	Set @name = Field(@row,'Firstname')
	Set @email = Field(@row,'EmailAddress')

next @i
```

