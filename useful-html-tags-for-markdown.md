---
author: me
title: useful-html-tags-for-markdown
created_date: 2026-04-08
type: reference
tags: [guide, markdown]
---

## abstract

collection of possibly useful html tags for advanced md usage.

## Intro

Markdown의 탄생 배경 자체가 **HTML을 더 읽고 쓰기 쉽게 만들기 위한 껍데기**이므로, HTML을 활용하여 더 다양하게 문서를 작성할 수 있다.

### HTML TAGS

#### 1. `<mark></mark>`

this is to <mark>hightlight</mark> sections.

```markdown
This part is very <mark>important</mark>.
```

#### 2. `<br>`

to make line break.

```markdown
This is first line. <br> This is second line.
```

#### 3. `<u></u>`

making <u>underline</u>

```markdown
This part is very <u>important</u>.
```

#### 4. `<span style="color: red"></span>`

- to change <span style="color: red">color</span> of texts.

```markdown 
This is changed <span style="color: red">color</span>.
```

- <span style="background-color: white; color: orange">background color</span> can be also changed<br> using`background-color: yellow`.

```markdown
<span style="background-color: white; color: orange">background color</span> can be also changed<br> using`background-color: yellow`.

<!-- use ';' to separate attributes -->
```

#### 5. `<details> <summary>`

- to make collapsable section.

```html
<details>
	<summary>open up details</summary>
	detail content.
</details>
```

- example:
<details>
	<summary>open up details</summary>
	detail content.
</details>

#### 6. `<sub> / <sup>`

- `2<sup>10</sup>` -> 2<sup>10</sup>
- `H<sub>2</sub>O` -> H<sub>2</sub>O

#### 7. `<center>`

#### 8. Codeblock

In case you want to display the plain code block that is not supposed to be executed such as `dataview`, then you can wrap around like below.

```markdown
~~~markdown
	```dataview
	```
~~~
```

both `~~~` and \`\`\` function as codeblock.

## Related

- [[Dataview-Note]]
- [[Base-inline-code]]
- [[syncthing-self-guide]]
