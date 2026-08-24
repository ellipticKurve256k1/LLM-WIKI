---
title: Base inline code guide
tags: [base, guide]
type: reference
---

# Base Inline Code Guide

## Abstract

Brief guide over Obsidian's core-plugin `Bases` as inline code. This provides simple starting code.

## Details

~~~markdown
```base
properties:
  file.name:
    displayName: Name
views:
  - type: table
    name: Default_View
    limit: 5
```
~~~

## Example Display

```base
properties:
  file.name:
    displayName: Name
  note.author:
    displayName: Author
  note.title:
    displayName: Title
views:
  - type: table
    name: Example_View
    filters:
      or:
        - file.tags.contains("log")
        - day_rating > 5
    order:
      - title
      - author
      - file.folder
      - file.ctime
    limit: 10
    columnSize:
      file.ctime: 201

```