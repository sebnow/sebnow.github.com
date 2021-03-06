---
layout: post
title: NeoVim - A Clean Slate
categories: [tech]
---

According to [Git][initial_commit], my vimrc is twelve years old, and
it's probably actually older. That's a long time to accrue a lot of
cruft. A lot has happened in the world of Vim in the past decade, with
the emergence of [NeoVim][neovim] encouraging the development of
features found in newer editors.  Simultaneously, tooling for languages
has improved significantly with the creation of the [Language Server
Protocol][lsp] (LSP), and newer languages, such as Rust, learning from
their predecessors and providing tooling out of the box. I think it's
time to wipe the slate clean and start anew, looking for modern
solutions to configure the editor the way I want it to work.

My main goal is to have functionality rivaling that of "modern" editors
and IDEs, while having access to the modes, keybindings and performance
of Vim. NeoVim 0.5 provides a fantastic foundation for achieving this,
with a native LSP client, Lua support, a built in terminal, and
asynchronous jobs.

To start off, I've removed all but the most [minimal
settings][clean_commit] to make NeoVim usable for me.

The ecosystem around the LSP client is still maturing, as 0.5 is yet to
be released, but it's starting to become viable as a daily driver.

  * [ncm2][] already includes support for it, which provides
completion-as-you-type for multiple sources (LSP, snippets,
omnicomplete).
  * [diagnostic-nvim][] improves the user experience by delaying
diagnostics while in insert mode - in most cases you don't end up with
a well-formed program with each key press - and utilizing floating
windows to present information in a more readable manner.
  * [nvim-lsp-denite][] providing integration with Denite.

Currently, the most mature alternative to the NeoVim LSP client is
[CoC][coc], which provides a great user experience, but tries to do too
much for my taste. It appears to be providing a bridge to VSCode
extensions.

I'm looking at using [Denite][denite] as a unified interface for quickly
navigating things like Code Actions, files, buffers, and git. It may
also serve as a platform to run common tasks such as running tests in
a consistent way. I haven't used it previously, mostly due to the
dependency on Python, but using it will remove the need for other
plugins such as [ctrlp.vim][], [fzf.vim][], [vim-clap][], etc.

I will be elaborating on specific parts of my set up in future posts.

[coc]: https://github.com/neoclide/coc.nvim
[ctrlp.vim]: https://github.com/kien/ctrlp.vim
[denite]: https://github.com/Shougo/denite.nvim
[diagnostic-nvim]: https://github.com/haorenW1025/diagnostic-nvim
[fzf.vim]: https://github.com/junegunn/fzf.vim
[initial_commit]: https://github.com/sebnow/configs/blob/b3dccd4a5dc025285f7ad2bee12bf7d4144b2104/.vimrc
[lsp]: https://microsoft.github.io/language-server-protocol
[ncm2]: https://github.com/ncm2/ncm2
[neovim]: https://neovim.io
[nvim-lsp-denite]: https://github.com/weilbith/nvim-lsp-denite
[vim-clap]: https://github.com/liuchengxu/vim-clap
[clean_commit]: https://github.com/sebnow/configs/blob/0a826279e957ca07b107bdb7563237ad133c10dc/vim/settings.vim
