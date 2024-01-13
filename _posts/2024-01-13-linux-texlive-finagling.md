---
title: "Finessing LaTeX installations on Linux"
author_profile: true
layout: single
---

## TeX Live
For as long as I've used Linux,[^1] I've used [TeX Live](https://tug.org/texlive/) as my TeX distribution.
The first time around, after consulting the internet (possibly [this very helpful thread](https://tex.stackexchange.com/questions/245982/differences-between-texlive-packages-in-linux)), I decided to just get everything I might ever need at once and installed `texlive-full`.
But that's criminally large, it took up 5 GB of space.[^2]
So the second time around,[^3] I decided to install something smaller.
Which brings us to the issue of installing the bits and pieces that eventually even the large and relatively comprehensive `texlive-latex-extra` lacks.[^4]

## The problem
There are a million ways the internet suggests one might install a single package.
Well, [this thread](https://tex.stackexchange.com/questions/73016/how-do-i-install-an-individual-package-on-a-linux-system) suggests four, anyways.

You might try to use `tlmgr`, which should be easy, because it's built in to TeX Live.
I tried it, but immediately ran into a wall of version incompatibilities.
Installing packages specifically through `apt` seems plausible, but that means finding the package and hoping it's not bundled with others I already have, creating chaos.
Manual installation looks convoluted and painful, and I would sooner give in to `texlive-full` than download and pass more `.sty` files around (if that would even work generally).

So, I decided to figure out why `tlmgr` wasn't working.[^5]
Looks like `apt`'s Tex Live is older than the most up-to-date version.
I didn't like the idea of trying to install TeX Live with something other than `apt`,[^6] so I ruled that out.
Then I found someone explaining how to tell `tlmgr` to use an older repository which matched the TeX Live version I have.[^7]
First, you need to add the right repository for your version,
```
tlmgr repository add ftp://tug.org/historic/systems/texlive/<year>/tlnet-final
```
then, set it as the default
```
tlmgr option repository ftp://tug.org/historic/systems/texlive/<year>/tlnet-final
```
I don't think you need to remove the old[^8] useless repository, but I did. 
You can see what it is with `tlmgr list` and remove it with `tlmgr repository remove <repo>`.

The only wrinkle here being you still need to know what the package is actually called, and that may well not be what you put in `\usepackage{}`.

[^1]: Well, as long as I've used it relatively seriously. We had a fling in high school, but I had even less idea what I was doing then than I do now. So, that is to say, not _that_ long.
[^2]: It didn't help that I made poor sizing choices when partitioning, and gave `/` a mere 20 GB that filled up in no time at all.
[^3]: When I finally stopped dragging my feet and updated to an OS that was going to be supported for more than another month or two.
[^4]: I will not surrender and take the easy way out without at least a bit of smashing my head into walls. Even though re-partitioned to give myself more working room.
[^5]: I know it was trying to _tell_ me, but errors make more sense after you've figured out the meaning than when you're staring at the output.
[^6]: I have had too many weird updating issues on this computer to be cavalier about adding more to the pile. The software updater has never worked quite right.
[^7]: It feels like this is something `tlmgr` should be able to figure out, if it knows the error's cause, but who am I to argue?
[^8]: Well, new, actually, but old as in "being replaced."