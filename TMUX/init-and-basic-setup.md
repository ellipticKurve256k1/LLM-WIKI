---
title: tmux basic init and setup
tags: [tmux, mac, zsh]
type: reference
priority: 2
finished: true
created_date: 2026-04-25
---

# TMUX BASIC SETUP AND INIT

## Abstract

Tmux basic installation guide and key binding for future reference.

### Basic Installation

**Recommended installation:** `use brew in MacOS`

```bash
brew install tmux
```

### Basic Key bindings

#### Init Tmux

```bash
tmux
```

#### Split window

#### Through Command

```bash
tmux split-window -h -p 30 // horizontally
tmux split-window -v -p 20 // vertically
```

#### Through Key bind

**Important Notes**:

- Tmux commands always start with `ctrl + b,`

##### split window vertically

```bash
ctrl + b, %
```

##### resize the right pane 

1. get into the command mode

```bash
ctrl + b, :
```

2. input the command

```bash
:resize-pane -R 35
```

2.1 navigations
   -  R, L, D, U 

#### Move the cursor between windows

```bash
ctrl + b, arrow <-, ->
```

#### close the window out of the splitted windows

```bash
ctrl + b, x // hit `y` when inquired
```

#### scroll within the pane out of splitted windows

1. get into `copy mode`

```bash
ctrl + b, [
```

2. use **arrow key** to scroll down and up
3. use `ctrl + c` to exit the copy mode

### Kill Session

1. kill current session

```bash
tmux kill-session
```

2. kill all sessions

```bash
tmux kill-server
``` 

3. ls sessions

```bash
tmux ls
```
