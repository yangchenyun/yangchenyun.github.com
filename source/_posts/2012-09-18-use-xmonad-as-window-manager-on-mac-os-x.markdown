---
layout: post
title: "use xmonad as window manager on mac OS X"
date: 2012-09-18 14:20
comments: true
categories: 
published: false
---

## Why Use Xmonad


## Install on Mac OS X

### Step 1 Build Haskell Env
Install the `Haskell` environment which **Xmonad** depends on:

    brew update
    brew install ghc haskell-platform


### Step 2 Install xmonad Packages
It gonna take a while. Once finished we need the package manager in `Haskell` to install any Haskell packages:

    brew install cabal-install

Now let's install **Xmonad** and its dependencies:

    cabal upgrade
    cabal install xmonad

`cabal` will solve the dependency and once it is installed, we have a working `xnomad` binary under `~/.cabal/bin`.

### Step 3 Configure X11 to load xmonad

Now we need to config `X11` to load xmonad when it is launched.

    mkdir ~/.xinitrc.d
    vim vim 90-xmonad.sh

    #! /bin/sh
    # "chmod +x ~/.xinitrc.d/90-xmonad.sh" to activate
    USERWM=$HOME/.cabal/bin/xmonad

Now you could start `X11` server with `xinit` and start the first [tour][Tour] under xnomad.

The `X11` or `XQuarz` will be opened, press `Command+Alt+A` to switch between you desktop and X11 server.

Next let start configuring our xnomad.

## Configure Xmonad

[tmpl]: http://www.haskell.org/haskellwiki/Xmonad/Config_archive/Template_xmonad.hs_%280.8%29
[config]: http://www.haskell.org/haskellwiki/Xmonad/General_xmonad.hs_config_tips
[tutorial]: http://thinkingeek.com/2011/11/21/simple-guide-configure-xmonad-dzen2-conky/

### Dzen2
Download [dzen2][dzen2], it is a tool to manage status bar

Use options 4 in the `config.mk` file

    LIBS = -L/usr/lib -lc -L${X11LIB} -lX11 -lXinerama -lXpm
    CFLAGS = -Os ${INCS} -DVERSION=\"${VERSION}\" -DDZEN_XINERAMA -DDZEN_XPM

    make install

### Install Xmonad-Contrib
`xmonad-contrib` is used to access 
When I try to `cabal install xmonad-contrib`, it complains about missing `pkg-config xfr`. So I directly put the dependency in the installment:

    cabal install xmonad-contrib --flags="-use_xft"

## Further Readings

[dzen2]: https://sites.google.com/site/gotmor/dzen2-latest.tar.gz?attredirects=0
[Tour]: http://xmonad.org/tour.html
[FAQ]: http://www.haskell.org/haskellwiki/Xmonad/Frequently_asked_questions#What_build_dependencies_does_xmonad_have.3F
