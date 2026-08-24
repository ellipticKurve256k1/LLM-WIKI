---
title: obsidian-vault-required-settings-checklist
type: reference
tags: [settings, documentation, reference, obsidian, checklist]
---

# Obsidian Vault Required Settings-checklist

## Definition

Checklist of baseline Obsidian vault settings to apply for a consistent working setup.

## Required Settings Checklist

### General

- [x] Install and enable the `Terminal` community plugin
- [x] Install and enable the `Calendar` community plugin
- [x] Enable Obsidian CLI support if available

### Appearance

- [x] Set theme to `PLN`
	- [x] Set font-size to `14`
	- [ ] Adjust [[obsidian-vault-required-settings-checklist#Custom Snippet for `PLN` theme|Custom Snippet Setting]]
- [ ] Set accent color to `RGB(247,147,26)` 
- [ ] Set text font to `Operator Mono`
- [ ] Set interface font to `Operator Mono`
- [ ] Hide the ribbon menu

### Shortcuts

- [ ] Set `Cmd + Shift + S` for left sidebar
- [ ] Set `Cmd + Shift + P` for right sidebar
- [ ] Set `Cmd + J` to open the `integrated terminal in the root directory`.
- [ ] Set `Cmd + D` for creating today's daily note
  - [ ] Remove the default `Delete paragraph` to avoid conflicts with `Cmd + D`
- [ ] Set `Cmd + Shift + E` for toggling Live Preview and Source mode
- [ ] Set `Cmd + R` for Reveal in Finder
- [ ] Set `Cmd + M` for rename file
- [ ] Set `Cmd + Shift + .` for `toggle blockquote`
- [ ] Set `Alt + p` for `toggle pin`

### Editor

- [ ] Set default view mode to `Reading View`
- [ ] Set default editing mode to `Source Mode`
- [ ] Enable Vim mode
- [ ] Show editing mode in the status bar
- [ ] Show `Line Numbers` under Display

### Files And Links

- [ ] Enable automatic updating of internal links

### Web Extension

- [ ] Install `Obsidian Web Clipper`

### ETC Reference

- Base inline code starter

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

### Custom Snippet for `PLN` theme

This snippet increases the default folder text size in the PLN theme.

1. Make a `snippets` folder inside `.obsidian` folder.
2. Inside the `snippets` folder, write `font-sidebar.css` css code below.
	- **IMPORTANT:** Do **not** use the default **Copy** button on the code block, as it may insert invalid whitespace characters. Instead, manually select the entire CSS snippet and copy-paste it.

```css
.nav-folder-title,
.nav-file-title {
  font-size: 12px;
}

.nav-folder-title,
.nav-file-title {
  line-height: 1.4;
}
```

3. Restart Obsidian, then check the **CSS snippets** section under **Appearance** to ensure your custom `.css` file is loaded and enabled.