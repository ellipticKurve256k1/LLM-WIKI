---
title: Useful Dataview plugin notes
author: me
type: guide
tags: [guide, reference, dataview]
---

# Useful `Dataview` plugin notes.

## Abstract

`Dataview` plugin is a wonderful tool to query notes using `frontmatter` [[Built-in-metadata|metadata]]. 
This is a collection of all `Dataview` plugin references for future use.
More details can be found at [official document](https://blacksmithgu.github.io/obsidian-dataview/)

### Table

`Table` can be very useful if I want to display custom notes in **table format**.

#### Examples

##### Display Table 1

Scenario: Display **3** items of title, description and link, where `frontmatter` metadata where `day_rating` is greater than 2.  

- **Tips**

	- Directory of file note can be copied by right clicking on one of the file in the directory and paste the `copy path from vault folder` 
	- metadata field must be used **without** quote, whereas if the value is string, it must contains quote. 
	- If don’t want to display ‘files’ default column, include `without ID`.

- Code[^1]:

~~~markdown
```dataview
table without ID
title as Title, description as Desc, file.link as Link 
from "Logs/DLogs/2026/03_March"
where day_rating > 2
limit 3
```
~~~

### List

`List` is similar to `Table`, but it only lists in bullet points.

~~~markdown
```dataview
list 
from "Logs/DLogs/2026/03_March"
where day_rating > 5
limit 3
```
~~~

### Tags

`tags` is **not** just regular custom metadata. It has built in feature in that cannot be used just like any other metadata.

~~~markdown
```dataview
table
from #daily_log  // when using #tags 'no directory' and should be 'from'
where day_rating > 5
limit 3
```
~~~

[^1]: reference in [[useful-html-tags-for-markdown#8. Codeblock]]