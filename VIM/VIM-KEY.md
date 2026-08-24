---
title: VIM KEY
tags:
  - vim
  - shortkeys
type: reference
priority: 3
finished: true
created_date: 2026-04-18
---

# VIM KEY 

## Abstract 

Very useful vim keys for reference.
And descriptions about `VIM`.
This is for my own notes that I organize. 

## Special Notes

VIM command consists of `operator` + `motion` / `object` 

- Example:
- `operator`: such as `d`, `c`.
- `motion`: such as `w` or `p` that selects the word or paragraph.

## Details

### Basic Operators

|operator|meaning|example|explanation|
|---|---|---|---|
|d|delete|dw|delete the word|
|c|change|cw|delete the word and get into  insert mode|
|y|yank|yw|yank the word|
|gu|convert to lower case|guiw|convert this current word to lower case| 
|gU|convert to upper case|gUiw|convert this current word to upper case| 
|g~|toggle the case of text|g~iw|toggle the case of the current word|
|>|indent|v>|indent current entire line|
|<|outdent|v<|outdent current entire line|
|v|visual mode|viw|select current word in visual mode| 

### Basic Motions

|Motion|meaning|example|
|---|---|---|
|`w`|move cursor to next word|none|
|`b`|move cursor prev word|none|
|`iw`|inside of word of current cursor position|`viw`|
|`ip`|inside of paragraph|`vip`|
|[[VIM-KEY#Quick Cursor move]]|none|none|
|[[VIM-KEY#Inside functionality]]|none|none|

#### Entering `visual` mode

- `v`: character wise  
- `V`: line wise

#### Entering insert mode

- `i`: entering insert mode before current cursor.
- `o`: entering insert mode, adds a **new line below** from the current line.
- `O`: entering insert mode, adds a **new line above** from the current line.
- `a`: entering insert mode, writing from the **next character** after the current cursor position.
- `A`: entering insert mode at the end of the line.

### Motions

#### Quick Cursor move

- `$`: move cursor to end of line. (right)
- `0`: move cursor to the left end. (left)

- `g_`: move to the last non-blank (non-whitespace) character on the line (right)
- `^`: move cursor to the the first non-blank (non-whitespace) character of the line (left)

- `gg`: move cursor top.
- `G`: move cursor bottom. 

- `w`: next word
- `b`: previous word

#### Navigation 

A number before a command repeats it. 
For example, `5h` moves left by 5 characters.

- `h`: left
- `j`: down
- `k`: up
- `l`: right

- `H`: move cursor to the top of the screen 
- `L`: move cursor to the bottom of the screen
- `M`: move cursor to the middle of the screen
- `zz`: recenters the screen so that cursor line becomes the middle line of the window.

### Copy and Paste

- `y`: yank(copy)
- `Y` or `yy`: yank the entire line
- `p`: paste
- `P`: paste 'before' cursor

#### Special copy features 

Normally, `yanking` only registers copied data to VIM registry. 
In order to actually save data to system clipboard, `"+` should be typed before yank.

**example**

- `"+y`
- `"+p`

**Notice**

Not all VIM environment offer such `moving data to system clipboard`, check before it has built-in system clipboard feature.

`vim --version | grep clipboard`

if `+clipboard` has `+` sign, then data can be copied to system clipboard.
if not, then re-install the vim may be necessary.

### Functionalities(Operators)

- `/`: search 
- `n`: find next word
- `N`: find prev word
- `u`: undo 
- `:e`: reload current file
- `ctrl + r`: forward from undo
- `:set number`: show line number
- `:set nonumber`: hide line number
- `:($number)`: move cursor to the `($number)` line
- `d`: delete something in **range** and keep it in `normal mode`
	- `operator` + `motion` combination
		- `dw`: delete word from current cursor position
		- `dd`: cut the entire line.
		- `d$`: delete entire line from the current cursor position 
- `c`: **change** operation. It deletes the current selected range and enters in `insert mode`
	- **change** is delete and insert mode
	    - `cw`: **change** the word from the current position  
		- `c$`: change entire line from the current cursor position 
- `r`: replace a character 
- `x`: delete by character
	- Both are act as `backspace`
    - `X`: delete the character preceeding cursor 
    - `ctrl + h`: delete the character preceeding cursor

#### Inside functionality

- **Inside**
	- `di` + `"`: delete **inside** of target while keeping `normal mode`
	- `ci` + `"`: change **inside** of target
	- **Target**
		- `"`
		- `(`
		- `[`
	- `vi` + `object`: select inside of `object`
		- ex: `vip` -> visual select inside paragraph.  
        - **paragraph: distinguished by white space**
    - `yi` + `object`: copy the inside of `object`
- `inside`: does not include internal space ex) `i`
- `around`: does include internal space ex) `a`

	- **Block parenthesis**
		Without explicitly typing actual target bracket, if the target is parenthesis.
        - `cib`: `b` does work as `block` that automatically find and deletes what ever next parenthesis block `(`.
        - `cab`: `around` command deletes including the next found parenthesis.
		
- **Indentation**
    - `>>`: indent right in `normal mode`
    - `>`: indent `selected area` in visual to right 

### Special Operator

- `.`: It repeats the last change you made
	- scope: `operator` + `motion` and edit word

> [!note]
> - `ggVGy`: copy the entire document
> 	- `gg`: move the cursor to the top
> 	- `V`: Visual mode (line visual)
> 	- `G`: move the cursor to the bottom
> 	- `y`: yank

### VIM File functionalities

- `:e $filename`: open a file 
- `double ctrl + w`: switch the cursor within the same terminal seprated window 

### `VSCODE` VIM plugin 

MacOS has its own feature called `Press and Hold` that if you hold one key it will not repeat inputing the key, it rather gets changed to different special character.

Thus, app like VSCode gets overwritten by `Press and Hold` settings. So installing VIM mode in VScode as plugin will not allow `Key Repeat`. So to enable key repeat, following command should be input in terminal.

```bash
defaults write com.microsoft.VSCode ApplePressAndHoldEnabled -bool false
```

### VIM Plugin

[[VIM-Plugin]]

#### VIM-Surround

![[VIM-Plugin#VIM-SURROUND]]

#### VIM-commentary 

```vim
gcc <!-- activate commentary --> 
```

#### VIM Tab

You can use many files in vim as tab and switch between them

```bash
:tabnew filename
```

- open up several files in tabs

```bash
vim -p filename1 filename2
```

**Switch between tabs**

- next tab

```bash
gt
```

- prev tab

```bash
gT
```

