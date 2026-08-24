---
title: VIM command note
tags:
  - vim
  - shortkeys
type: reference
priority: 3
finished: true
created_date: 2026-04-19
modified_date: 2026-04-19
---

# VIM command note

## Abstract

Organized note for understanding what each VIM command part means and how the commands shown in the reference can be combined.

## Key Points

- Some keys are single actions such as `d`, `c`, `y`, `p`, and `x`.
- Some keys act as movement or range parts such as `w`, `b`, `$`, `0`, `gg`, and `G`.
- Combined commands in this note are limited to the forms already shown in `VIM-Reference.md`.

## Details

### Single command meanings

- `w`: next word
- `b`: previous word
- `$`: move cursor to the right end
- `0`: move cursor to the left end
- `gg`: move cursor top
- `G`: move cursor bottom
- `d`: delete something in range and keep `normal mode`
- `c`: change operation; deletes the selected range and enters `insert mode`
- `y`: yank(copy)
- `Y`: yank the entire line
- `p`: paste
- `P`: paste before cursor
- `x`: delete by character
- `v`: character wise visual mode
- `V`: line wise visual mode
- `i`: enter insert mode before current cursor
- `a`: enter insert mode from the next character after current cursor
- `o`: enter insert mode with a new line below
- `O`: enter insert mode with a new line above

### Range and target parts used in combinations

- `w` in `dw` and `cw`: word range from the current cursor position
- `d` in `dd`: delete line action used twice for the entire line
- `i` in `di"` and `ci"`: inside
- `"` in `di"` and `ci"`: quote target
- `(` and `[` can also be targets for inside operations
- `b` in `cib` and `cab`: block parenthesis target

### Combination forms

- `dw`: delete word from current cursor position and stay in `normal mode`
- `dd`: cut the entire line
- `cw`: change the word from the current position and enter `insert mode`
- `di"`: delete inside quotes and keep `normal mode`
- `ci"`: change inside quotes
- `cib`: change inside the next parenthesis block
- `cab`: change around the next parenthesis block, including the parenthesis

### Combination examples already shown

- `d` + `w` -> `dw`
- `d` + `d` -> `dd`
- `c` + `w` -> `cw`
- `d` + `i` + `"` -> `di"`
- `c` + `i` + `"` -> `ci"`
- `c` + `i` + `b` -> `cib`
- `c` + `a` + `b` -> `cab`

### Full command sequence example

- `ggVGy`: copy the entire document
- `gg`: move cursor to the top
- `V`: visual mode(line visual)
- `G`: move cursor to the bottom
- `y`: yank
