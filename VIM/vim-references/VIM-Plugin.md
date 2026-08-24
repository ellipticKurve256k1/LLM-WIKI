---
title: VIM Plugin
tags: [reference, vim, plugin]
type: reference
priority: 3
finished: true
created_date: 2026-04-19
---

# VIM Plugin 

## Abstract

Reference about VIM Plugin

### Basic Plugin Settings   

**Install `vim-plug`**

```bash
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

### Deleting Plugin

1. Delete the plugin line in `.vimrc`
2. `:PlugClean!`

### Updating Plugin

`:PlugUpdate`

### Plugin  

#### VIM-TERRAFORM

1. Installation

```vim
Plug 'hashivim/vim-terraform'
```

2. Configuration at `vimrc`

```vim
" terraform vim plugin setting
let g:terraform_fmt_on_save = 1
let g:terraform_align = 1
filetype plugin indent on
autocmd FileType terraform setlocal shiftwidth=2 tabstop=2 softtabstop=2 expandtab
```

#### VIM-SURROUND

1. go to `~/.vimrc`

```vim
call plug#begin('~/.vim/plugged')
Plug 'tpope/vim-surround'
call plug#end()
```

2. Execute the installation in `vim`

```vim
:PlugInstall
```

3. Method

Example: surrounding **selecting word in visual** with `"`

```vim
S"
```

#### NERDTree

**installation** 

```vim
Plug 'preservim/nerdtree'
```

**Toggle**

```vim
// commands
:NERDTree

// hotkeys
`R` -> refresh NERDTree directory
```

#### FZF

```vim
:Files   // search files, need to install file binaries if inquired.
```

#### GitGutter

- Enabling GitGutter or Refresh the git status

```vim
:GitGutter
```

- View GitGutter Hunk Preview

```vim
:GitGutterPreview
```

- closing Preview 

1. move cursor to that panel using `ctrl + w + w` and `:q`
    - this also can move around up and down to read full changes

2. use `:pclose` command 

- this is closing `preview` window.

```vim
:pclose
```

- move to next hunk

```vim
]+c
```

- move to prev hunk

```vim
[+c
```

#### Fugitive

- everything starts with `:Git` and use as same as you would use `git cli`

#### Current Plugin Setup

```vim

call plug#begin('~/.vim/plugged')
Plug 'tpope/vim-surround'
Plug 'tpope/vim-commentary'
Plug 'vim-airline/vim-airline'
Plug 'vim-airline/vim-airline-themes'
Plug 'preservim/nerdtree'
Plug 'junegunn/fzf' 
Plug 'junegunn/fzf.vim'
Plug 'airblade/vim-gitgutter'
Plug 'tpope/vim-fugitive'
call plug#end()

```

#### VIM current theme and settings

```vim

set shiftwidth=2  
set number
set autoread
set noswapfile
colorscheme gruvbox
set background=dark
set t_Co=256
set updatetime=1000
set signcolumn=yes
set ignorecase
autocmd FocusGained,BufEnter,CursorHold * checktime

```

#### ETC

- ignoring case sensitivity when finding

```vim
:set ignorecase
```

#### VIM theme settings(gruvboxp theme)

1. Make `colors` folder under `.vim` directory.
2. `git clone https://github.com/morhetz/gruvbox.git ~/.vim/pack/colors/start/gruvboxp`
3. set configuration. see `VIM current theme and setting` as reference

** Explanation **
- set autoread: auto reflect changes made by externally
- set t_Co=256: only if term supports 256 colors, can be checked by `echo $TERM`
- set noswapfile: do not create swap file 
