---
title: Obsidian DataView Docs
date: "2021-07-12 07:59"
tags:
  - obsidian
  - dataview
public: false
---

# Obsidian DataView Docs

# Introduction

Dataview is an advanced query engine / index for generating dynamic views of the data in your knowledge base. You can
collect pages by tags, folders, contents, or *fields*, which are arbitrary values associated with pages. A typical
page using dataview might look something like this:

````
# Daily Retrospective

#daily

Date:: 2020-08-15
Rating:: 7.5
Woke Up:: 10:30am
Slept:: 12:30am
````

If you have many such pages, you could easily create a table via

````
table date, rating, woke-up, slept FROM #daily
````

which would produce nice looking table like so:

![](https://blacksmithgu.github.io/obsidian-dataview/assets/images/daily-retro-example-table-f48c123df504f19bc640133a0b9f5295.png)

You could then filter this view to show only days with a high rating; or sort days by their rating or the time you woke
up, and so on so forth.

# Pages and Fields

The core data abstraction for dataview is the *page*, corresponding to a markdown page in your vault with associated
*fields*. A *field* is just a piece of arbitrary named data - text, dates, durations, links - which dataview
understands, can display prettily, and can filter on. Fields can be defined in three ways:

1. **Frontmatter**: All YAML frontmatter entries will automatically be converted into a dataview field.
1. **Inline Fields**: A line of the form `<Name>:: <Value>` will automatically be parsed by dataview as a field. Note
   that you can surround `<Name>` with standard Markdown formatting, which will be discarded.
1. **Implicit**: Dataview annotates pages with a large amount of metadata automatically, like the day the file was
   created, any associated dates, links in the file, tags, and so on.

An example page with associated fields created using both methods may be:

````
---
duration: 4 hours
reviewed: false
---
# Movie X

**Thoughts**:: It was decent.
**Rating**:: 6
````

### Field Types

Dataview understands several different field types:

* **Text**: The default catch-all. If a field doesn't match a more specific type, it is just plain text.
* **Number**: Numbers like '6' and '3.6'.
* **Boolean**: true/false, as the programming concept.
* **Date**: ISO8601 dates of the general form `YYYY-MM[-DDTHH:mm:ss]`. Everything after the month is optional.
* **Duration**: Durations of the form `<time> <unit>`, like `6 hours` or `4 minutes`. Common english abbreviations like
  `6hrs` or `2m` are accepted.
* **Link**: Plain Obsidian links like `[[Page]]` or `[[Page|Page Display]]`.
* **List**: Lists of other dataview fields. In YAML, these are defined as normal YAML lists; for inline fields, they are
  just comma-separated lists.
* **Object**: A map of name to dataview field. These can only be defined in YAML frontmatter, using the normal YAML
  object syntax:
  ````
  field:
    value1: 1
    value2: 2
    ...
  ````

The different field types are important for ensuring Dataview understands how to properly compare and sort values, and
enable different operations.

### Implicit Fields

Dataview automatically adds a large amount of metadata to each page:

* `file.name`: The file title (a string).
* `file.folder`: The path of the folder this file belongs to.
* `file.path`: The full file path (a string).
* `file.link`: A link to the file (a link).
* `file.size`: The size (in bytes) of the file (a number).
* `file.ctime`: The date that the file was created (a date + time).
* `file.cday`: The date that the file was created (just a date).
* `file.mtime`: The date that the file was last modified (a date + time).
* `file.mday`: The date that the file was last modified (just a date).
* `file.tags`: An array of all tags in the note. Subtags are broken down by each level, so `#Tag/1/A` will be stored in
  the array as `[#Tag, #Tag/1, #Tag/1/A]`.
* `file.etags`: An array of all explicit tags in the note; unlike `file.tags`, does not include subtags.
* `file.inlinks`: An array of all incoming links to this file.
* `file.outlinks`: An array of all outgoing links from this file.
* `file.aliases`: An array of all aliases for the note.

If the file has a date inside its title (of form `yyyy-mm-dd` or `yyyymmdd`), or has a `Date` field/inline field, it also has the following attributes:

* `file.day`: An explicit date associated with the file.

# Creating Queries

Once you've added useful data to relevant pages, you'll want to actually display it somewhere or operate on it. Dataview
allows this through `dataview` code blocks and inline queries, where you can write queries/code and have them be dynamically executed and
displayed in the note preview. For writing such queries, you have three options:

1. The dataview [query language](/docs/query/queries) is a simplistic, SQL-like language for quickly creating views. It
   supports basic arithmetic and comparison operations, and is good for basic applications.
1. The query language also provides inline queries, which allow you to embed single values
   directly inside a page - for example, todays date via `= date(today)`, or a field from another page via `= [[Page]].value`.
1. The dataview [JavaScript API](/docs/api/intro) gives you the full power of JavaScript and provides a DSL for pulling
   Dataview data and executing queries, allowing you to create arbitrarily complex queries and views.
1. Similar to the query language, you can write JS inline queries, which let you embed a computed JS value directly. For
   example, `$= dv.current().file.mtime`.

The query language tends to lag in features compared to the JavaScript API, primarily since the JavaScript API lives
closer to the actual code; the counter-argument to this fact is that the query language is also more stable and is less
likely to break on major Dataview updates.

### Using the Query Language

You can create a query language dataview block in any note using the syntax:

````
```dataview
... query ...
```
````

The details of how to write a query are explained in the [query language documentation](/docs/query/queries); if you learn
better by example, take a look at the [query examples](/docs/query/examples).

### Using Inline Queries

You can use an inline query via the syntax

````
`= <query language expression>`
````

where the expression is written using the [query language expression language](/docs/query/expressions). You can
configure inline queries to use a different prefix (like `dv:` or `~`) in the Dataview settings.

### Using the JavaScript API

You can create a JS dataview block in any note using the syntax:

````
```dataviewjs
... js code ...
```
````

Inside of a JS dataview block, you have access to the full dataview API via the `dv` variable. For an explanation of
what you can do with it, see the [API documentation](/docs/api/code-reference), or the [API
examples](/docs/api/code-examples).

### Using JavaScript Inline Queries

You can use a JavaScript inline query via the syntax

````
`$= <js query language expression>`
````

You have access to the `dv` variable, as in `dataviewjs` codeblocks, and can make all of the same calls. The result
should be something which evaluates to a JavaScript value, which Dataview will automatically render appropriately.

# Queries

The dataview query language is a simple, structured, custom query language for quickly creating views on your data. It
supports:

* Fetching pages associated with tags, folders, links, and so on.
* Filtering pages / data by simple operations on fields, like comparison, existence checks, and so on.
* Sorting results based on fields.

The query language supports the following view types, described below:

1. **TABLE**: The traditional view type; one row per data point, with several columns of field data.
1. **LIST**: A list of pages which match the query. You can output a single associated value for each page.
1. **TASK**: A list of tasks whose pages match the given query.

## General Format

The general format for queries is:

````
```dataview
TABLE|LIST|TASK <field> [AS "Column Name"], <field>, ..., <field> FROM <source> (like #tag or "folder")
WHERE <expression> (like 'field = value')
SORT <expression> [ASC/DESC] (like 'field ASC')
... other data commands
```
````

Only the 'select' statement (describing what view and what fields) is required. If the `FROM` statement is omitted, the
query runs automatically over all markdown pages in your vault. If other statements (like `WHERE` or `SORT`) are
present, they are run in the order they are written. Duplicate statements are allowed (multiple `WHERE` statement, for eaxmple).

* For the different view types, only the first line (the 'select' section, where you specify the view type and fields to
  display) differs. You can apply *data commands* like *WHERE* and *SORT* to any query, and you can select from any
  [source](/docs/query/sources) using *FROM*.

See [expressions](expressions) for context on what expressions are, and [sources](sources) for context on what sources are.

## Query Types

### List Queries

Lists are the simplest view, and simply render a list of pages (or custom fields) which match the query.
To obtain a list of pages matching the query, simply use:

````
LIST FROM <source>
````

For example, running `LIST FROM #games/moba or #games/crpg` might show:

![List Example](data:image/png;base64,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)

You can render a single computed value in addition to each matching file, by adding an expression after `LIST`:

````
LIST <expression> FROM <source>
````

For example, running `LIST "File Path: " + file.path FROM "4. Archive"` might show:

![List Example](https://blacksmithgu.github.io/obsidian-dataview/assets/images/file-path-list-e10d7b78c544b424ae316520bdcde1e8.png)

### Table Queries

Tables support tabular views of page data. You construct a table by giving a comma separated list of the YAML frontmatter fields you want to render, as so:

````
TABLE file.day, file.mtime FROM <source>
````

You can choose a heading name to render computed fields by using the `AS` syntax:

````
TABLE (file.mtime + dur(1 day)) AS next_mtime, ... FROM <source>
````

An example table query:

````
TABLE time-played AS "Time Played", length as "Length", rating as "Rating" FROM #game
SORT rating DESC
````

![Table Example](https://blacksmithgu.github.io/obsidian-dataview/assets/images/game-0417d5136353f57fcdb8903b1dcc9a2b.png)

### Task Queries

Task views render all tasks whose pages match the given predicate.

````
TASK from <source>
````

For example, `TASK FROM "dataview"` might show:

![Task Example](https://blacksmithgu.github.io/obsidian-dataview/assets/images/project-task-3a548a1a3f88cd4e4d97f9582433f86d.png)

## Data Commands

The different commands that dataview queries can be made up of. Commands are
executed in order, and you can have duplicate commands (so multiple `WHERE`
blocks or multiple `GROUP BY` blocks, for example).

### FROM

The `FROM` statement determines what pages will initially be collected and passed onto the other commands for further
filtering. You can select from any [source](/docs/query/sources), which currently means by folder, by tag, or by incoming/outgoing links.

* **Tags**: To select from a tag (and all its subtags), use `FROM #tag`.
* **Folders**: To select from a folder (and all its subfolders), use `FROM "folder"`.
* **Links**: You can either select links TO a file, or all links FROM a file.
  * To obtain all pages which link TO `[[note]]`, use `FROM [[note]]`.
  * To obtain all pages which link FROM `[[note]]` (i.e., all the links in that file), use `FROM outgoing([[note]])`.

You can compose these filters in order to get more advanced sources using `and` and `or`.

* For example, `#tag and "folder"` will return all pages in `folder` and with `#tag`.
* `[[Food]] or [[Exercise]]` will give any pages which link to `[[Food]]` OR `[[Exercise]]`.

### WHERE

Filter pages on fields. Only pages where the clause evaluates to `true` will be yielded.

````
WHERE <clause>
````

1. Obtain all files which were modified in the last 24 hours:

````
LIST WHERE file.mtime >= date(today) - dur(1 day)
````

2. Find all projects which are not marked complete and are more than a month old:

````
LIST FROM #projects
WHERE !completed AND file.ctime <= date(today) - dur(1 month)
````

### SORT

Sorts all results by one or more fields.

````
SORT date [ASCENDING/DESCENDING/ASC/DESC]
````

You can also give multiple fields to sort by. Sorting will be done based on the first field. Then, if a tie occurs, the second field will be used to sort the tied fields. If there is still a tie, the third sort will resolve it, and so on.

````
SORT field1 [ASCENDING/DESCENDING/ASC/DESC], ..., fieldN [ASC/DESC]
````

### GROUP BY

Group all results on a field. Yields one row per unique field value, which has 2 properties: one corresponding to the field being grouped on, and a `rows` array field which contains all of the pages that matched.

````
GROUP BY field
GROUP BY (computed_field) AS name
````

In order to make working with the `rows` array easier, Dataview supports field "swizzling". If you want the field `test` from every object in the `rows` array, then `rows.test` will automatically fetch the `test` field from every object in `rows`, yielding a new array.
You can then apply aggregation operators like `sum()` over the resulting array.

### FLATTEN

Flatten an array in every row, yielding one result row per entry in the array.

````
FLATTEN field
FLATTEN (computed_field) AS name
````

For example, flatten the `authors` field in each literature note to give one row per author:

````
table authors from #LiteratureNote
flatten authors
````

![Flatten Example](https://blacksmithgu.github.io/obsidian-dataview/assets/images/flatten-authors-ae602de33e28e0a4d66977d134774e54.png)

# Expressions

Dataview query language *expressions* are anything that yields a value - all fields are expressions, as are literal
values (like `6`), as are computed values (like `field - 9`). For a very high level summary:

````
# General
field               (directly refer to a field)
simple-field        (refer to fields with spaces/punctuation in them like "Simple Field!")
a.b                 (if a is an object, retrieve field named 'b')
a[expr]             (if a is an object or array, retrieve field with name specified by expression 'expr')
f(a, b, ...)        (call a function called `f` on arguments a, b, ...)

# Arithmetic
a + b               (addition)
a - b               (subtraction)
a * b               (multiplication)
a / b               (division)

# Comparison
a > b               (check if a is greater than b)
a < b               (check if a is less than b)
a = b               (check if a equals b)
a != b              (check if a does not equal b)
a <= b              (check if a is less than or equal to b)
a >= b              (check if a is greater than or equal to b)

# Special Operations
[[Link]].value      (fetch `value` from page `Link`)
````

More detailed explanations of each follow.

## Expression Types

### Fields as Expressions

The simplest expression is one that just directly refers to a field. If you have a field called "field", then you can
refer to it directly by name - `field`. If the field name has spaces, punctuation, or other non-letter/number
characters, then you can refer to it using Dataview's simplified name, which is all lower case with spaces replaced with
"-". For example, `this is a field` becomes `this-is-a-field`; `Hello!` becomes `hello`, and so on.

### Arithmetic

You can use standard arithmetic operators to combine fields: addition (`+`), subtraction (`-`), multiplication (`*`),
and division (`/`). For example `field1 + field2` is an expression which computes the sum of the two fields.

### Comparisons

You can compare most values using the various comparison operators: `<`, `>`, `<=`, `>=`, `=`, `!=`. This yields a
boolean true or false value which can be used in `WHERE` blocks in queries.

### Array/Object Indexing

You can retrieve data from arrays via the index operator `array[<index>]`, where `<index>` is any computed expression.
Arrays are 0-indexed, so the first element is index 0, the second element is index 1, and so on.  For example `list(1, 2, 3)[0] = 1`.

You can retrieve data from objects (which map text to data values) also using the index operator, where indexes are now
strings/text instead of numbers. You can also use the shorthand `object.<name>`, where `<name>` is the name of the value
to retrieve. For example `object("yes", 1).yes = 1`.

### Function Calls

Dataview supports various functions for manipulating data, which are described in full in the [functions
documentation](functions). They have the general syntax `function(arg1, arg2, ...)` - i.e., `lower("yes")` or
`regexmatch("text", ".+")`.

---

## Type-specific Interactions & Values

Most dataview types have special interactions with operators, or have additional fields that can be retrieved using the
index operator.

### Dates

You can retrieve various components of a date via indexing: `date.year`, `date.month`, `date.day`, `date.hour`,
`date.minute`, `date.second`, `date.week`. You can also add durations to dates to get new dates.

### Durations

Durations can be added to each other or to dates. You can retrieve various components of a duration via indexing:
`duration.years`, `duration.months`, `duration.days`, `duration.hours`, `duration.minutes`, `duration.seconds`.

### Links

You can "index through" a link to get values on the corresponding page. For example `[[Link]].value` would get the value
`value` from page `Link`.

# Sources

A dataview "source" is something that identifies a set of files, tasks, or other data object. Sources are indexed internally by
Dataview, so they are fast to query. Dataview currently supports three source types:

1. **Tags**: Sources of the form `#tag`.
1. **Folders**: Sources of the form `"folder"`.
1. **Links**: You can either select links TO a file, or all links FROM a file. 

* To obtain all pages which link TO `[[note]]`, use `[[note]]`. 
* To obtain all pages which link FROM `[[note]]` (i.e., all the links in that file), use `outgoing([[note]])`.

You can compose these filters in order to get more advanced sources using `and` and `or`. 

* For example, `#tag and "folder"` will return all pages in `folder` and with `#tag`. 
* `[[Food]] or [[Exercise]]` will give any pages which link to `[[Food]]` OR `[[Exercise]]`.

Sources are used in both the [FROM query statement](/docs/query/queries#from), as well as various JavaScript API query calls.

# Functions

Dataview functions provide more advanced ways to manipulate data.

## Function Vectorization

Most functions can be applied either to single values (like `number`, `string`, `date`, etc.) OR to lists of those
values. If a function is applied to a list, it also returns a list after the function is applied to each element
in the list. For example:

````
lower("YES") = "yes"
lower(list("YES", "NO")) = list("yes", "no")

replace("yes", "e", "a") = "yas"
replace(list("yes", "ree"), "e", "a") = list("yas", "raa")
````

## Constructors

Constructors which create values.

### `object(key1, value1, ...)`

Creates a new object with the given keys and values. Keys and values should alternate in the call, and keys should
always be strings/text.

````
object() => empty object
object("a", 6) => object which maps "a" to 6
object("a", 4, "c", "yes") => object which maps a to 4, and c to "yes"
````

### `list(value1, value2, ...)`

Creates a new list with the given values in it.

````
list() => empty list
list(1, 2, 3) => list with 1, 2, and 3
list("a", "b", "c") => list with "a", "b", and "c"
````

### `date(any)`

Parses a date from the provided string, date, or link object, if possible, returning null otherwise.

````
date("2020-04-18") = <date object representing April 18th, 2020>
date([[2021-04-16]]) = <date object for the given page, refering to file.day>
````

### `number(string)`

Pulls the first number out of the given string, returning it if possible. Returns null if there are no numbers in the
string.

````
number("18 years") = 18
number(34) = 34
number("hmm") = null
````

### `link(path, [display])`

Construct a link object from the given file path or name. If provided with two arguments, the second argument is the
display name for the link.

````
link("Hello") => link to page named 'Hello'
link("Hello", "Goodbye") => link to page named 'Hello', displays as 'Goodbye'
````

### `elink(url, [display])`

Construct a link to an external url (like `www.google.com`). If provided with two arguments, the second
argument is the display name for the link.

````
elink("www.google.com") => link element to google.com
elink("www.google.com", "Google") => link element to google.com, displays as "Google"
````

---

## Numeric Operations

### `round(number, [digits])`

Round a number to a given number of digits. If the second argument is not specified, rounds to the nearest whole number;
otherwise, rounds to the given number of digits.

````
round(16.555555) = 7
round(16.555555, 2) = 16.56
````

--

## Objects, Arrays, and String Operations

Operations that manipulate values inside of container objects.

### `contains(object|list|string, value)`

Checks if the given container type has the given value in it. This function behave slightly differently based on whether
the first argument is an object, a list, or a string.

* For objects, checks if the object has a key with the given name. For example,
  ````
  contains(file, "ctime") = true
  contains(file, "day") = true (if file has a date in its title, false otherwise)
  ````

* For lists, checks if any of the array elements equals the given value. For example,
  ````
  contains(list(1, 2, 3), 3) = true
  contains(list(), 1) = false
  ````

* For strings, checks if the given value is a substring (i.e., inside) the string.
  ````
  contains("hello", "lo") = true
  contains("yes", "no") = false
  ````

### `extract(object, key1, key2, ...)`

Pulls multiple fields out of an object, creating a new object with just those fields.

````
extract(file, "ctime", "mtime") = object("ctime", file.ctime, "mtime", file.mtime)
extract(object("test", 1)) = object()
````

### `sort(list)`

Sorts a list, returning a new list in sorted order.

````
sort(list(3, 2, 1)) = list(1, 2, 3)
sort(list("a", "b", "aa")) = list("a", "aa", "b")
````

### `reverse(list)`

Reverses a list, returning a new list in reversed order.

````
reverse(list(1, 2, 3)) = list(3, 2, 1)
reverse(list("a", "b", "c")) = list("c", "b", "a")
````

### `length(object|array)`

Returns the number of fields in an object, or the number of entries in an array.

````
length(list()) = 0
length(list(1, 2, 3)) = 3
length(object("hello", 1, "goodbye", 2)) = 2
````

### `sum(array)`

Sums all numeric values in the array

````
sum(list(1, 2, 3)) = 6
````

### `all(array)`

Returns `true` only if ALL values in the array are truthy. You can also pass multiple arguments to this function, in
which case it returns `true` only if all arguments are truthy.

````
all(list(1, 2, 3)) = true
all(list(true, false)) = false
all(true, false) = false
all(true, true, true) = true
````

### `any(array)`

Returns `true` if ANY of the values in the array are truthy. You can also pass multiple arguments to this function, in
which case it returns `true` if any of the arguments are truthy.

````
any(list(1, 2, 3)) = true
any(list(true, false)) = true
any(list(false, false, false)) = false
all(true, false) = true
all(false, false) = false
````

### `none(array)`

Returns `true` if NONE of the values in the array are truthy.

### `join(array)`

Joins elements in an array into a single string (i.e., rendering them all on the same line). If provided with a second
argument, then each element will be separated by the given separator.

````
join(list(1, 2, 3)) = "1, 2, 3"
join(list(1, 2, 3), " ") = "1 2 3"
join(6) = "6"
join(list()) = ""
````

---

## String Operations

### `regexmatch(pattern, string)`

Checks if the given string matches the given pattern (using the JavaScript regex engine).

````
regexmatch("\w+", "hello") = true
regexmatch(".", "a") = true
regexmatch("yes|no", "maybe") = false
````

### `regexreplace(string, pattern, replacement)`

Replaces all instances where the *regex* `pattern` matches in `string`, with `replacement`. This uses the JavaScript
replace method under the hood, so you can use special characters like `$1` to refer to the first capture group, and so on.

````
regexreplace("yes", "[ys]", "a") = "aea"
regexreplace("Suite 1000", "\d+", "-") = "Suite -"
````

### `replace(string, pattern, replacement)`

Replace all instances of `pattern` in `string` with `replacement`.

````
replace("what", "wh", "h") = "hat"
replace("The big dog chased the big cat.", "big", "small") = "The small dog chased the small cat."
replace("test", "test", "no") = "no"
````

### `lower(string)`

Convert a string to all lower case.

````
lower("Test") = "test"
lower("TEST") = "test"
````

### `upper(string)`

Convert a string to all upper case.

````
upper("Test") = "TEST"
upper("test") = "TEST"
````

## Utility Functions

### `default(field, value)`

If `field` is null, return `value`; otherwise return `field`. Useful for replacing null values with defaults. For example, to show projects which haven't been completed yet, use `"incomplete"` as their defualt value:

````
default(dateCompleted, "incomplete")
````

Default is vectorized in both arguments; if you need to use default explicitly on a list argument, use `ldefault`, which
is the same as default but is not vectorized.

````
default(list(1, 2, null), 3) = list(1, 2, 3)
ldefault(list(1, 2, null), 3) = list(1, 2, null)
````

### `choice(bool, left, right)`

A primitive if statement - if the first argument is truthy, returns left; otherwise, returns right.

````
choice(true, "yes", "no") = "yes"
choice(false, "yes", "no") = "no"
choice(x > 4, y, z) = y if x > 4, else z
````

### `striptime(date)`

Strip the time component of a date, leaving only the year, month, and day. Good for date comparisons if you don't care
about the time.

````
striptime(file.ctime) = file.cday
striptime(file.mtime) = file.mday
````

# Examples

A small collection of simple usages of the dataview query language.

---

Show all games in the game folder, sorted by rating, with some metadata:

````
```dataview
TABLE time-played, length, rating FROM "games"
SORT rating DESC
```
````

![Game Example](https://blacksmithgu.github.io/obsidian-dataview/assets/images/game-0417d5136353f57fcdb8903b1dcc9a2b.png)

---

List games which are MOBAs or CRPGs.

````
```dataview
LIST FROM #game/moba or #game/crpg
```
````

![Game List](data:image/png;base64,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)

---

List all tasks in un-completed projects:

````
```dataview
TASK FROM #projects/active
```
````

![Task List](https://blacksmithgu.github.io/obsidian-dataview/assets/images/project-task-3a548a1a3f88cd4e4d97f9582433f86d.png)

---

List all of the files in the `books` folder, sorted by the last time you modifed the file:

````
```dataview
TABLE file.mtime FROM "books"
SORT file.mtime DESC
```
````

---

List all files which have a date in their title (of the form `yyyy-mm-dd`), and list them by date order.

````
```dataview
LIST file.day WHERE file.day
SORT file.day DESC
```
````
