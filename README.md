Emacs Build Scripts
===================

These are the emacs build scripts that produces the builds at
http://emacsformacosx.com/.

Prerequisites
-------------

### Hardware Requirements

The scripts are modular and are designed to be run on multiple build
machines (or VMs) and integrate with continuous integration servers (the
builds on emacsformacosx.com run from Jenkins now). This means that you can
build whatever architectures you have access to.

Note that cross-compiling Emacs is (still) not possible due to the "unexec"
step, which requires the binary that was built to be run. So if you want to
build an old architecture (like PowerPC), you need to be running on a system
that can actually execute binaries of that architecture.


### XZ

Recent Emacs pretests are being distributed in `.tar.xz` format. The
"fetch-emacs-from-ftp" script will convert from `.xz` to `.tar.bz2` so that
XZ doesn't need to be installed on every build machine. But you will need
the "xz" program on the machines that runs "fetch-emacs-from-ftp". The
easiest way to get it is through [homebrew](http://brew.sh/): "brew install xz"


### XCode Command Line Tools

Building emacs requires that the XCode command line tools be installed so that
some libraries (libxml2, at least) are available.

    xcode-select --install

Usage
-----

There are 3 scripts that are designed to be run from some sort of Continuous
Integration software (the builds on http://emacsformacosx.com run from
Jenkins). All three scripts know the `--verbose` command, and are nice and
loud when it is given.

### fetch-emacs-from-ftp

This takes an ftp url (`ftp://ftp.gnu.org/gnu/emacs/`, for example), and
downloads the latest version of the Emacs source code found there. It will
also convert the source from a `.tar.xz` to a `.tar.bz2` (so that the main
build VMs don't need to have "XZ" installed).

### build-emacs-from-tar

This is the main build script. It takes a tar file and a "kind" (`pretest`,
`nightly`, or `release`) as input and unpacks the tar, builds it for a
single architecture, and tars up the resulting Emacs.app file.

Builds of the main Emacs source repository are expected to be packaged up
into tars elsewhere. http://emacsformacosx.com has a Jenkins job that pulls down
the latest code and then tars it up like so:

    DATE=$(date "+%Y-%m-%d_%H-%M-%S")
    SHORT=$(git rev-parse --short HEAD)
    DIR=emacs-$DATE-$SHORT
    git archive --prefix="$DIR/" HEAD | tar x
    (cd $DIR && ./autogen.sh)
    tar cjf $DIR.tar.bz2 $DIR


### combine-and-package

This takes multiple tar files as input, unpacks and combines them into a
final "fat" Emacs.app, then creates a final disk image (`.dmg`). It takes an
optional `--sign` parameter (`--sign="my identity"`) which makes it code
sign the Emacs.app.

Example
-------

    $ ./fetch-emacs-from-ftp -v ftp://ftp.gnu.org/pub/gnu/emacs
    curl --continue-at - --silent -O ftp://ftp.gnu.org/pub/gnu/emacs/emacs-28.1.tar.xz
    $ /usr/local/bin/xzcat emacs-28.1.tar.xz
    $ ls *.bz2
    emacs-28.1.tar.bz2
    $ ./build-emacs-from-tar -v -j 8 emacs-28.1.tar.bz2 release
      ... Lots out output snipped ...
    Built Emacs-28.1-11-x86_64.tar.bz2, Emacs-28.1-11-x86_64-extra-source.tar
    
### Running Emacs.app from the command line

Create shell emacs script with:

    $ nano /usr/local/bin/emacs
```
#!/bin/sh
/Applications/Emacs.app/Contents/MacOS/Emacs "$@"
```

 And ec script:
 
    $ nano /usr/local/bin/ec
```
#!/bin/sh
which osascript > /dev/null 2>&1 && osascript -e 'tell application "Emacs" to activate'
emacsclient -c "$@"
```
Create alias for emacsclient:

    $ ln -s /Applications/Emacs.app/Contents/MacOS/bin/emacsclient /usr/local/bin
Add "alias emacsclient=ec" to  ".zshrc".

License
-------

Copyright © 2004-2021 David Caldwell <david@porkrind.org>

The scripts and programs contained in this distribution are licensed under
the GNU General Public License (v3.0). See the LICENSE file for details.
