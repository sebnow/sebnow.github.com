---
layout: post
title: A Clearer View of the NeoVim Ecosystem
categories: [tech]
---

A while back I posted about [rewriting my Vim config from scratch]({%
post_url 2020-05-13-NeoVim-A-Clean-Slate %}) and focusing on using
Neovim. In that time, my assumptions about what plugins I'd be using
have changed, as I have discovered more plugins that are implemented in
Lua and depend on NeoVim 0.5+.

## Completion

I checked out [completion-nvim][] when I first looked into using the
built in LSP client, however it seemed to lack a few features I wanted,
like multiple sources. [ncm2][] was better at the time. Since then,
`completion-nvim` has added a slew of features to be on par with `ncm2`,
at least for me.

## Fuzzy Search

Instead of [denite][], I will now be looking into customizing
[telescope.nvim][]. It appears to provide similar functionality but is
written in Lua, which makes it faster than a Python implementation, and
for me makes it more hackable. It has LSP integration out of the box, so
that reduces the amount of work I have to do.

I'm currently using it as a fuzzy file finder (CtrlP/fzf replacement),
a buffer list/switcher, LSP references, and grep/rg.

## TreeSitter

Along with LSP and Lua, the most exciting feature being developed in
NeoVim is [TreeSitter][] integration. TreeSitter is an incremental
parser generator, initially used in Atom and GitHub. It allows NeoVim to
work on an actual abstract syntax tree instead of relying on regular
expressions. Highlighting can be more accurate, reliable,
and tolerant of errors. Code can be navigated contextually. Folds can
work on custom parser queries. A lot of cool stuff is made possible by
this. Max Brunsfeld, the author, has an excellent
[presentation](https://www.youtube.com/watch?v=Jes3bD6P0To) from
StrangeLoop 2018.

## Plugin Management

While I haven't started using it yet, [Packer][] looks like an
interesting replacement for `vim-plug`. Support for dependencies could
be a useful feature not available in `vim-plug`.

[ncm2]: https://github.com/ncm2/ncm2
[completion-nvim]: https://github.com/nvim-lua/completion-nvim
[denite]: https://github.com/Shougo/denite.nvim
[telescope.nvim]: https://github.com/nvim-lua/telescope.nvim
[TreeSitter]: https://tree-sitter.github.io/
[Packer]: https://github.com/wbthomason/packer.nvim
