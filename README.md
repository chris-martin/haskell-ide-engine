# Haskell IDE Engine
[![License BSD3][badge-license]][license]

[badge-license]: https://img.shields.io/badge/license-BSD3-green.svg?dummy
[license]: https://github.com/haskell/haskell-ide-engine/blob/master/LICENSE


This project aims to be __the universal interface__ to __a growing number of Haskell tools__, providing a __full-featured and easy to query backend__ for editors and IDEs that require Haskell-specific functionality.

__We are currently focusing on using the [Language Server Protocol](https://github.com/Microsoft/language-server-protocol/blob/master/protocol.md) as the interface via which 
we talk to clients.__

- [Haskell IDE Engine](#haskell-ide-engine)
    - [Features](#features)
    - [Installation](#installation)
        - Installation with [stack](#installation-with-stack) or [Nix](#installation-with-nix)
        - [ArchLinux](#archlinux)
    - [Editor Integration](#editor-integration)
        - Using HIE with [VS Code](#using-hie-with-vs-code), [Sublime Text](#using-hie-with-sublime-text), [Neovim](#using-hie-with-neovim) or [Atom](#using-hie-with-atom)
    - [Hoogle Docs on hover](#hoogle-docs-on-hover)
    - [Contributing](#contributing)
        - [Planned Features](#planned-features)
        - [This is *not* yet another `ghc-mod` or `ide-backend` project](#this-is-not-yet-another-ghc-mod-or-ide-backend-project)
        - [It's time to join the project!](#its-time-to-join-the-project)
        - [Development](#development)
    - [Architecture](#architecture)
        - [1. BIOS layer](#1-bios-layer)
        - [2. Plugin layer](#2-plugin-layer)
        - [3. IDE interfacing layer](#3-ide-interfacing-layer)
    - [Documentation](#documentation)

## Features
 
 - Supports plain GHC projects, cabal projects(sandboxed and non sandboxed) and stack projects
 - Fast due to caching of compile info
 - Uses LSP, so should be easy to integrate with a wide selection of editors
 - Diagnostics via hlint and GHC warnings/errors
   
   ![Diagnostics](http://i.imgur.com/1vqm4eF.gif)

 - Code actions and quick fixes via apply-refact
   
   ![Apply Refact](http://i.imgur.com/dBrSi5F.gif)

 - Type information and documentation(via hoogle) on hover

   ![Hover](http://i.imgur.com/AcvYROv.gif)

 - Jump to definition
   
   ![Find Def](http://i.imgur.com/kmCU2Bz.gif)

 - List all top level definitions

   ![Doc Symbols](http://i.imgur.com/GErcYqp.gif)

 - Highlight references in document
   
   ![Doc Highlight](http://i.imgur.com/YLjHs2s.gif)
   
 - Completion
   
   ![Completion](http://i.imgur.com/wR6IJ7M.gif)

 - Formatting via brittany

   ![Formatting](http://i.imgur.com/cqZZ8HC.gif)

 - Renaming via HaRe

   ![Renaming](http://i.imgur.com/z03G2a5.gif)
 
## Installation 

Both methods build HIE from the source code, so first,

```bash
$ git clone https://github.com/haskell/haskell-ide-engine
$ cd haskell-ide-engine
```

### Installation with stack

To install HIE

#### For GHC 8.2.2

```bash
stack install
```

#### For GHC 8.2.1

```bash
stack --stack-yaml=stack-8.2.1.yaml install
```

#### For GHC 8.0.2

```bash
stack --stack-yaml=stack-8.0.2.yaml install
```

### Installation with Nix

Alternatively, given that you have Nix installed:

```bash
$ stack install --nix
```

### ArchLinux

An [haskell-ide-engine-git](https://aur.archlinux.org/packages/haskell-ide-engine-git/) package is available on the AUR.

Using [Aura](https://github.com/aurapm/aura):

```
$ aura -A haskell-ide-engine-git
```

## Editor Integration

All of the editor integrations assume that you have already installed HIE (see above) and that `stack` put the `hie` binary in your path (usually `~/.local/bin` on linux and macOS).

### Using HIE with VS Code

Install from
[the VSCode marketplace](https://marketplace.visualstudio.com/items?itemName=alanz.vscode-hie-server), or manually from the repository [vscode-hie-server](https://github.com/alanz/vscode-hie-server).

### Using HIE with Sublime Text

* Make sure HIE is installed (see above) and directory stack put the `hie` binary in is in your path
  * (usually `~/.local/bin` on unix)
* Install [LSP](https://packagecontrol.io/packages/LSP) using [Package Control](https://packagecontrol.io/)
* From Sublime Text, press Command+Shift+P and search for Preferences: LSP Settings
* Paste in these settings. Make sure to change the command path to your `hie`

```
"clients": {
  "haskell-ide-engine": {
    "command": ["hie", "--lsp"],
    "scopes": ["source.haskell"],
    "syntaxes": ["Packages/Haskell/Haskell.sublime-syntax"],
    "languageId": "haskell",
  },
},
```

Now open a haskell project with Sublime Text. You should have these features available to you:

1. Errors are underlined in red
2. LSP: Show Diagnostics will show a list of hints and errors
3. LSP: Format Document will prettify the file

### Using HIE with Neovim

As above, make sure HIE is installed. Install and load the neovim plugin
[LanguageClient](https://github.com/autozimu/LanguageClient-neovim). If you use
[vim-plug](https://github.com/junegunn/vim-plug), then you can do this by e.g.
including the following line in the Plug section of your `init.vim`:

```
Plug 'autozimu/LanguageClient-neovim', { 'do': ':UpdateRemotePlugins' }
```

and issuing a `:PlugInstall` command within neovim.

Finally, make sure that `hie` is included as the language server source for haskell: 

```
let g:LanguageClient_serverCommands = {
    ...
    \ 'haskell': ['hie', '--lsp'],
    ...
    \ }
```

For asynchronous auto-completion, follow the setup instructions on
[LanguageClient](https://github.com/autozimu/LanguageClient-neovim).

### Using HIE with Atom

Make sure HIE is install, then install the two Atom packages [atom-ide-ui](https://atom.io/packages/atom-ide-ui) and [ide-haskell-hie](https://atom.io/packages/ide-haskell-hie),

```bash
$ apm install atom-ide-ui ide-haskell-hie
```

## Hoogle Docs on hover

HIE supports fetching docs from hoogle on hover. It can use the Hoogle database for 
your user, or a project specific database.

To generate a global hoogle database for your user from hackage, run

```bash
$ hoogle generate
```

To generate a global database for a stack installation, run

```bash
$ stack exec hoogle generate
```

To generate a project specific database for a stack project, run

```bash
$ stack hoogle
$ stack build
```

To generate a project specific database for a sandboxed cabal project, run

```bash
$ cabal install --enable-documentation --haddock-hoogle
$ hoogle generate --database=hiehoogledb.hoo --local=.
```
in the root dir of your cabal project

If you don't have the dependencies built with haddock/documentation enabled, you need 
to remove the sandbox and build everything again before running the commands
given above

```bash
$ cabal sandbox delete
$ cabal sandbox init
```

To generate a project specific database for a non-sandboxed cabal project, run

```bash
$ cabal install
$ hoogle generate --local --database=hiehoogledb.hoo
```

If HIE detects a project specific hoogle DB, it will use it over the global db.

## Contributing

### Planned Features

 - [x] Multiproject support
 - [ ] Project wide references
 - [ ] Cross project find definition
 - [ ] New-build support
 - [ ] HaRe refactorings
 - [ ] More code actions
 - [ ] Cross project/dependency Find Definition
 - [ ] Case splitting, type insertion etc.

### This is *not* yet another [`ghc-mod`](https://github.com/kazu-yamamoto/ghc-mod) or [`ide-backend`](https://github.com/fpco/ide-backend) project

 > Both the ghc-mod and ide-backend maintainers have agreed to contribute code to this new repository and then rebase the old repos on this. The reason we're using a new repo instead of modifying one of the existing ones is so that the existing projects experience no disruption during this migration process. If this was a new set of people starting a new project without support from existing projects, I'd agree with you. But Alan's reached out to existing players already, which is an important distinction.

This project is not started from scratch:

1. See why [we should supersede previous tools](/docs/Challenges.md)
2. Check the [list of existing tools and functionality](/docs/Tools.md)
3. See more [other tools and IDEs for inspiration](/docs/Inspirations.md)


### It's time to join the project!

:heart: Haskell tooling dream is near, we need your help! :heart:

 - Register in [our google group mailing list](https://groups.google.com/forum/#!forum/haskell-ide).
 - Join [our IRC channel](http://webchat.freenode.net/?channels=haskell-ide-engine) at `#haskell-ide-engine` on `freenode`.
 - Fork this repo and hack as much as you can.
 - Ask @alanz or @hvr to join the project.

### Development

Apart from stack you need [cask](https://cask.readthedocs.org/en/latest/) for the emacs tests. You can install it using

```
curl -fsSL https://raw.githubusercontent.com/cask/cask/master/go | python
```


## Architecture

Below you find a short overview of the main architectural layers of `haskell-ide-engine`.
For more info have a look in [the docs folder](/docs) at the root of this project, especially:

 - The [Architecture discussion](docs/Architecture.md)
 - The [Protocol discussion](docs/Protocol.md)
 - The [Design discussion](docs/Design.md)

### 1. BIOS layer

[`ghc-mod`](https://github.com/kazu-yamamoto/ghc-mod) stays an AGPL project,
and is used for its "awesome sauce" in terms of
the BIOS functions that it does so well. This interface is
[straightforward to use](http://alanz.github.io/haskell%20refactorer/2015/10/02/ghc-mod-for-tooling),
and if a license-constrained user wants to do something else it is also easy to
replace, if there is strong control of the operating environment.

### 2. Plugin layer

A layer providing a point to integrate tools and existing functions, probably
including ghci.

### 3. IDE interfacing layer

The focus is currently on [LSP](https://github.com/Microsoft/language-server-protocol/blob/master/protocol.md) as the protocol we use to interface with IDEs.

Existing transports are still functional for the time being.

## Documentation

All the documentation is in [the docs folder](/docs) at the root of this project.
