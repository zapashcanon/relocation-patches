Relocation Patches
==================

This is a set of patches to make some OPAM packages relocatable within
different OPAM switches. It has been designed to be used automatically
by (opam-bin)[https://github.com/OCamlPro/opam-bin] to patch packages
on the fly.

Scripting
---------

The `scripts/` directory contains a small set of scripts to help
create and test patches for relocation. Before using them, you must
first create an `env.sh` file in the top directory, similar to this
one:

```
#!/bin/bash

OPAMBIN_SRCDIR=$HOME/GIT/opam-bin/
OPAM_REPO=$HOME/GIT/opam-repository
PATCHES_DIR=$HOME/GIT/relocation-patches
```

where all these directories have been created by:

```
mkdir $HOME/GIT
cd $HOME/GIT
git clone git@github.com:OCamlPro/opam-bin
git clone git@github.com:ocaml/opam-repository
git clone git@github.com:OCamlPro/relocation-patches
```

Most of these scripts work by creating a `root/` subdirectory
containing an opam repository with switches `test1` and `test2`.

Here is how to use these scripts:

* `./scripts/test-all.sh` is used to test that a set of packages are
relocatable. It contains a list of packages to be tested. You should
modify these lists: `OK_PACKAGES` is a set of packages that are known
to be relocatable and should be installed beforehand. `PACKAGES` is
the set to be tested: for each package in the list, it will be
installed, moved to another switch, and tested by building the next
packages. The list should be ordered in dependency order. To test the
your package is relocatable, you should put it in the list, plus
packages that use it.

* `./scripts/prepare-package.sh NAME VERSION` is used to work on a particular
package NAME.VERSION that is not relocatable. It will create a
directory `work/NAME/VERSION/` containing: `OPAM`, its package
description, `VERSION.patch`, if a patch is already available,
`NAME.VERSION-orig` the original sources (that should not be
modified), `NAME.VERSION-reloc` the modified sources for relocation
(only different if a patch was found) and `NAME.VERSION-test`, a copy
of `NAME-VERSION-reloc` to test the patch. It will also create local
links to:

  * `./make-patch.sh` will update `VERSION.patch` by a diff between
  `NAME.VERSION-orig` and  `NAME.VERSION-reloc`
  * `./publish-patch.sh` will copy `VERSION.patch` back in the local
  copy of the `relocation-patches` directory
  * `./reapply-patch.sh` will regenerate  `NAME.VERSION-reloc` and
 `NAME.VERSION-test` from  `NAME.VERSION-orig` and `VERSION.patch`.
 It can be used to cleanup `NAME.VERSION-test` to test a new patch.

* `./scripts/to-install.sh` will read the files `pre-install.txt` and
  `to-install.txt` to generate a file `solution.txt` for
  `./scripts/test-solution.txt`. `pre-install.txt` contains a list of
  packages with versions that have to be in all solutions.
  `install.txt` contains a list of packages (with or without versions)
  that should be installed. The script will iter on all these packages
  to provide an installation for every one in a `solutions/`
  directory. It will then merge all the solutions in a file, and call
  opam to find a global solution with all the packages. It might fail
  if no solution exist: you will then have to either fix a package
  version in `pre-install.txt`, or remove completely a package from
  `install.txt` until a solution is found. Once the solution is OK,
  you can use `./scripts/test-solution.txt` to test that the solution
  seems relocatable.

How to make packages relocatable
--------------------------------

Most of the sources of errors with relocatable packages are:

* including a path to a filename in the sources (usually, a .ml file
  generated by ./configure or Makefile). The solution is to provide a patch
  that will use OPAM_SWITCH_PREFIX to recompute the path instead of using
  the hardcoded value. This is usually found in build/develpment tools
  like ocamlfind, ocamlbuild, etc.

* hardcoded dependencies using autolink, without other references to the
  library. These ones appear usually as a stubs library not found. The
  solution is to modify the META file to include an explicit dependency on
  the file (and ideally to remove the autolink reference). If the package
  installs files in a non-standard path, a solution is to move them back
  to the standard path (<$OPAM_SWITCH_PREFIX/lib/$PACKAGE_NAME>)
