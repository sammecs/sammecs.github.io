---
layout: post
title: '"Cannot execute binary file" Error while building Derivation'
---
This is a quickie.

## Problem description

While building a derivation, I get this error:
```bash
/nix/store/ds7r5c4z1vzrm0jw7ay712zs59p64hf7-stdenv-linux/setup: line 488: source: /nix/store/6lvv07wxgwi6641jkqbb11s20qdk3b7f-pa_stable_v190600_20161030.tgz: cannot execute binary file
```
For me, this was a weird error because changing `unpackPhase` or `preConfigurePhase` or any other attribute of the derivation didn't help me debug the problem (I thought there was an error in the unpack phase).

## Problem solution

In the file described in the error message, I can find the specified line. It reads `source "$pkgs"`, and is inside of the `activatePackage` function. It turns out that `activatePackage` is called by `_activatePkgs`, and `_activatePkgs` is always executed. `_activatePkgs` seems to activate the dependencies of a given derivation (which means adding executables to `$PATH` and maybe a bit more through setup hooks), and if the dependency is a file (`if [ -f "$pkg" ];`), then it is assumed that the right thing to do to activate the package is to source it. I think this is a bit counterintuitive, but at least the error is easy to fix: I wrongly supplied a file (I tried to use the `src` attribute of a derivation, foolishly thinking it would give me access to development headers when in reality it is just a `.tgz` archive) instead of a meaningful derivation to the dependencies of my package.
