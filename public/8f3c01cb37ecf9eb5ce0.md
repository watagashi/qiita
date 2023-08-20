---
title: MacVim-KaoriYa から MacVim + vim-kaoriya に移行する
tags:
  - Vim
  - macvim
private: false
updated_at: '2020-12-07T01:09:19+09:00'
id: 8f3c01cb37ecf9eb5ce0
organization_url_name: null
slide: false
---
MacVim-KaoriYa から MacVim + vim-kaoriya に移行してみました。

-   [splhack/macvim-kaoriya: MacVim-KaoriYa](https://github.com/splhack/macvim-kaoriya)
-   [MacVim by macvim-dev](https://macvim-dev.github.io/macvim/)
-   [koron/vim-kaoriya: Vim+kaoriya build system](https://github.com/koron/vim-kaoriya)

# 方針

-   MacVim と vim-kaoriya の中身を変更しない。
-   MacVim-KaoriYa 固有の `vimrc` 設定を極力反映する。
-   UTF-8 と CP932 でエンコーディングされたファイルを変換不要にする。
-   macOS 以外でも極力影響のない `vimrc` にする。

# 手順

## MacVim と vim-kaoriya のインストール

```bash
brew install macvim
test -d ~/.vim || mkdir ~/.vim
git clone https://github.com/koron/vim-kaoriya.git ~/.vim/vim-kaoriya
```

必要に応じてご利用のプラグインマネージャーに合わせ、別途  [vim-jp/vimdoc-ja: A project which translate Vim documents into Japanese.](https://github.com/vim-jp/vimdoc-ja) を導入して下さい。

## `vimrc` と `gvimrc` の作成

`~/.vim` に以下のように `vimrc_kaoriya.vim` と `gvimrc_kaoriya.vim` を作成します。

```vim:.vim/vimrc_kaoriya.vim
scriptencoding utf-8 " vim:set ts=8 sts=2 sw=2 tw=0 ff=unix fenc=utf-8:

" Mainly for MacVim without KaoriYa
" !git clone https://github.com/koron/vim-kaoriya.git ~/.vim/vim-kaoriya

if has('kaoriya')
  finish
endif

let s:vimfiles = isdirectory(expand('~/vimfiles'))
      \ ? expand('~/vimfiles') : expand('~/.vim')
let s:kaoriyavimdirectory = s:vimfiles . '/vim-kaoriya/kaoriya/vim'

if !filereadable(s:kaoriyavimdirectory . '/vimrc')
  echomsg 'vim-kaoriya not found.'
  finish
endif

let s:vim = $VIM
let $VIM = s:kaoriyavimdirectory
execute 'silent! source ' . $VIM . '/vimrc'
let $VIM = s:vim
unlet s:vim

" Enforces the character encoding used by Vim internally to UTF-8.
" Because vim-kaoriya/kaoriya/vim/switches/catalog/utf-8.vim
" is not working well.
set encoding=utf-8
set fileencodings=ucs-bom,utf-8,iso-2022-jp-3,euc-jisx0213,euc-jp,cp932

set ambiwidth=double
if has('osxdarwin')
  set printmbfont=r:HiraMinProN-W3,b:HiraMinProN-W6
endif

" Reset &term
if has('unix') && !has('gui_running')
  set term=$TERM
endif

```

```vim:.vim/gvimrc_kaoriya.vim
scriptencoding utf-8 " vim:set ts=8 sts=2 sw=2 tw=0 ff=unix fenc=utf-8:

" Mainly for MacVim without KaoriYa
" !git clone https://github.com/koron/vim-kaoriya.git ~/.vim/vim-kaoriya

if has('kaoriya')
  finish
endif

let s:vimfiles = isdirectory(expand('~/vimfiles'))
      \ ? expand('~/vimfiles') : expand('~/.vim')
let s:kaoriyavimdirectory = s:vimfiles . '/vim-kaoriya/kaoriya/vim'

if !filereadable(s:kaoriyavimdirectory . '/vimrc')
  " echomsg 'vim-kaoriya not found.'
  finish
endif

let s:vim = $VIM
let $VIM = s:kaoriyavimdirectory
execute 'source ' . $VIM . '/gvimrc'
let $VIM = s:vim
unlet s:vim

if has('gui_macvim')
  set guifont=Osaka-Mono:h14
  set noimdisable
endif
```

作った Vim スクリプトを有効にするため、 `~/.vim/vimrc` と `~/.vim/gvimrc` に以下を追加します (先頭近くが良いと思います)。

```vim:vimrc
source <sfile>/vimrc_kaoriya.vim
```

```vim:gvimrc
source <sfile>/gvimrc_kaoriya.vim
```


# 課題

-   ISO-2022-JP でエンコーディングされたファイルは `:e ++enc=iso-2022-jp` のようにエンコーディングの指定が必要。
-    macOS のターミナルで CUI 版を実行したとき一瞬 `E558` のエラーが出る。  
    ![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/127827/a4be21c0-ad09-06c7-158b-55eabdbf803c.png)

ほかにもあるかも。
