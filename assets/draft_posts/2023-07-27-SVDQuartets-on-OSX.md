---
title: "Homebrew and GSL"
author_profile: true
layout: single
---

Today in "weird compilation notes that someone may eventually find helpful": installing SVDquartets on a Mac, but not the PAUP* version.

First, Homebrew has chosen to put GSL somewhere the makefiles can't find.
Before trying to make anything, open the makefiles and amend the lines with `GSLLIBDIR` and `INCL_GSL`.
Assuming you have already installed GSL via homebrew and linked it and just can't remember _where_ it is, if you try to link it again it will tell you, run `brew link gsl`.
Then take that whole path, which in my case was `/opt/homebrew/Cellar/gsl/2.7.1` and amend the above lines to, `GSLLIBDIR = /opt/homebrew/Cellar/gsl/2.7.1/lib` and `INCL_GSL = /opt/homebrew/Cellar/gsl/2.7.1/include`.

Then I got an error about the use of the `time()` function, which turned out to be an issue of `time.h` not being included in quartet_gen.c.

After that, things should work.