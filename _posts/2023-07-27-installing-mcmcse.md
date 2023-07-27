---
title: "Installing mcmcse on a Mac"
author_profile: true
layout: single
---

For reasons not worth going into, I wanted to install the R package `mcmcse`.
This became a pain thanks to its dependency `fftwtools`, so I didn't do it for weeks.
For posterity, and in case it helps anyone else, here's what I had to do to install `fftwtools` on a Mac (an M1 Mac in particular).

(I seem to recall difficulties of a similar nature when I wanted to get `mcmcse` working on Ubuntu as well.)

## TL;DR
This is really just a note on ways to point R to things it doesn't know where to look for or can't find.
The short version is: you can run `system("PATH=/path/to/whatever:${PATH}")` to add something to the `PATH` from inside an R session just long enough for R to find it and install it.
If R needs that every time, I do not have the solution here, but it probably involves the `.Renviron` file.

## Tip: disable auto updating
This saga will require, potentially, running `brew install`.
For reasons I cannot fathom, this auto updates things.
Because I'm lazy, I installed R with homebrew, which means when it auto-updates packages, my _entire goddamn R installation_ gets shot to hell.
The umpteenth time this happened and I got the equivalent of "new R who dis?" when I went to load packages I used daily, I figured out how to make it stop.
Now I only call homebrew with the `HOMEBREW_NO_AUTO_UPDATE=1` flag enabled, as `HOMEBREW_NO_AUTO_UPDATE=1 brew install <whatever>`.
More on this [here](https://computingforgeeks.com/prevent-homebrew-auto-update-on-macos/).

## Overcoming `pkg-config` errors
The first error I encountered was that `pkg-config` could not be found.
Assuming that you have it installed already (I did, otherwise you want `HOMEBREW_NO_AUTO_UPDATE=1 brew install pkg-config`), the solution can be found [here](https://stackoverflow.com/questions/70599640/can-i-change-the-location-of-homebrew-fftw-install-r-cant-seem-to-read-fftw3-h) which links to information [here](https://stackoverflow.com/questions/32403262/adding-path-to-rstudio-s-path/32403524#32403524) which outsources some explanation [here](https://stuff.mit.edu/afs/sipb/project/r-project/lib/R/library/base/html/Startup.html).

The short version: somewhere you need to get the place `pkg-config` is added to your `PATH`.

I tried creating `.Renviron` to set this for R in perpetuity, but it didn't seem to work.
I probably should have figured out why, but instead I opted for a bandaid to just get the damn thing to install, and ran `system("PATH=/opt/homebrew/bin:${PATH}")` in R.

That worked, and revealed another problem.

## You need `fftw`
The package `fftwtools` is an interface to the library `fftw`, so you need to have that.
If you don't, run `HOMEBREW_NO_AUTO_UPDATE=1 brew install fftw`.

## Victory is life
Congratulations, you can now install `mcmcse`.
Have fun!