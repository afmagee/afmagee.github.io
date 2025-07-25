---
title: "Bicycles and the art of software design"
author_profile: true
layout: single
---

For the last several years of my professional life, one big theme has been the modularity and generality of code.
I've written code since 2013, and I've been (a small) part of a few large collaborative projects for Bayesian inference tools since 2017.
I've even dabbled in making my own [tools from scratch](https://github.com/afmagee/treess).[^1]
It wasn't until then, I think, that I really started to appreciate that organizing code is both an art and really, really important.
Or, as someone recently summed up for me in a conversation about academics and code standards, a lot of academic code is more _script_ than _software_.[^2]
Since a lot of scientists don't get much of any teaching in software design, and can generally get away without it, we often don't get exposed to good design principles.

## Stretching the limits of analogies

A common comment I've seen (and, goodness knows, made) when adjusting from a "write the script" mindset to a "design some software" mindset is something like, "it's hard for me to follow what's going on if I don't see the steps all laid out in order."
And, you know, I'm sympathetic.
Reading _everything_ that happens, in order, is like reading an essay, or a manuscript.
And it can indeed be faster to figure out than the many interconnected, non-linearly-organized, chunks of code in a better-designed codebase.
But it's also exactly the wrong way to write something that you're going to work with in any extended way.[^3]
In the long run, you want to be able to write for people to see the big picture quickly, not the details.

Now, for the tortured analogy: bikes.
When you get on a bike, and you want to go, you just pedal, and it goes.
One might say that `bike.go()` has a simple, high-level interface.
But a bike is not made to go by one large do-everything bit, pedals turn the cranks, which pull the chain, which in turn moves the back wheel.
When one part of this system needs to be worked on, or replaced, it can be done separately from the others.
One big thing that does everything is hard to maintain, and it's hard to extend.
Both of these are just variations on fiddling with how some piece works, really, but they're doing so for different enough purposes that they can be different in practice.

But I don't intend this to be a book-like blog post, so, drivetrains are too large a topic for more specific examples.
So instead we will consider one component: the integrated brake and shifter lever.
These exist for mountain bikes and hybrids, but they've completely taken over road bikes, where they've acquired the name "brifter" (br(eak + shi)ifter).
If you've had a road bike made since about the early aughts, you've probably had them on your bike.
They're not one thing that does everything, but they're one thing that does two things, so, good enough for a thoroughly eviscerated metaphor.

Most of the time, brifters are pretty great.
You click, they shift, you pull, they brake.
Everything you need to do to adjust your speed is there in one place and easy to work with.
The problems start when the brifters break, or when you decide you want to change something about your gearing.
Since we're going to talk tradeoffs, we'll need to ask what the alternatives are.
There are [quite a few](https://www.sheldonbrown.com/upgrade-gears.html), but the common theme is "separate levers for shifting and braking."[^4]

## Everybody likes to build, nobody likes to maintain

Here's a list of reasons I can recall I've had to rewrite code that I thought of as relatively stable.
It is not exhaustive.
In fact, it's all from one project.[^5]
And not a project where I was just writing scripts, one where my colleagues and I were trying to write relatively modular, maintainable code.
- I was having MCMC trouble and I suddenly needed to configure more sampler settings from elsewhere in the codebase.
- The maintainers of a dataset archive shifted from saving daily copies to monthly copies.
- I went to run an analysis and everything went to hell in a handbasket.

There's also the old classic, the maintainers of a software package you depend on changed how something worked, and now nothing works.[^6]

The point is, a lot of unexpected things can lead to code breaking.
Undiscovered bugs and edge cases are common, but sometimes things far beyond our control will change.
So what are we to do?
If we develop the perfectly maintainable codebase, then nothing does much of anything, and it's not particularly _useful_.
On the other end of the spectrum we have one big script, one line after another, in which case if _anything_ changes, you incur a large amount of time required to fix it all back up.
Clearly there's a trade-off, the art is managing to fall somewhere on the spectrum that works for a particular project.

Okay, back to bikes.
Brifters are less repairable than they are simply disposable.
When they malfunction, sometimes you get lucky and squirting something in to clean them out makes them work again.
(Just like sometimes you get lucky and changing one function call fixes your broken script.)
Otherwise, it's time to replace the whole unit.
And now you're paying extra maintenance time costs, because you're not just replacing a shift lever, you're replacing the brake lever too.
It may not quite be twice the work, but it's twice the fiddly bits you then have to dial back into working right (a process not entirely unlike debugging).


## Tinker, tantrum, throw in the towel

Aside from maintaining code when things go wrong, another common experience is adding or changing functionality.
Extending it.
This can happen because you, the coder, get curious about something.
It can happen when you talk to someone you're collaborating with on the project and _they_ get curious about something.
It can happen years later when you think, "you know what, this project will be easy, because I can just dust off that script from a while back."
And if none of those lead to you needing to extend your code, at least one reviewer comment probably will.

The problem is, when you've just written a script, this can be a real pain, much like maintaining it.
(Because, unlike mechanical breakdowns, code breakdowns happen when something changes, and so fixes also involve changes.)
Compared to more modular code, one big script has a lot more hard-coded assumptions about what, exactly, the objects at hand are.[^8]
This means assumptions about how to act on them, and all sorts of things that can need to be changed if you want to add an option.
Those assumptions accrue, and compound, to the point where one small change can topple the whole kit and caboodle.

Back to brifters.
What does "extending" mean?
Probably changing your gearing.
Say, to get a wider range.
Maybe so you can get into touring, or tackle a high pass, or maybe you're just moving somewhere hillier.
So, maybe you choose to go from a double chain ring to a triple,[^9] as the less-invasive change[^10].
Well, once again, like broken brifters, since they're one piece, a change that mucks with your shifting, now means you're replacing your perfectly good brake levers too.


## So, was there a point to all this?[^11]

Now, at this point, I've complained to hell and back about do-it-all scripts, analogized them to brifters, and in so doing slandered the latter.
Which leads to some questions, like, "why am I still reading this madman's ramblings?"[^12] or, if you're feeling charitable, "then, why are there brifters?"[^13]
And the answer is, because they work, and they're convenient.
When maintained, they give you slick, easy shifting from two different hand positions,[^14] and that's just nice.
Especially since one of those is where you usually keep your hands, and most bikes that most people ride have both shifters and brakes available there.
The alternatives are easier to maintain, or extend, but they do come with a learning cost, much like a better organized code base.

And, the thing is, sometimes one script that does it all _is_ the right answer.
Sometimes, code is straightforward enough and simple enough that the thing to do is just to do it, modularity be damned.
The hard thing is knowing when that is, and when it's time to invest in something a bit more fleshed out.
And, of course, it depends.
It depends on your audience, and how far they're willing to go to learn how to use something.
It depends on how done things really are, or, at least, how done they'll be when they're done.[^7]
Which may not be a satisfying answer, but it's the truth.
We'll never get anywhere if we try to make everything the best possible maintainable extensible form.
But we'll be too busy rewriting everything to see where we are if we never bother to engineer things vaguely optimally in the first place.
Or, to paraphrase George Box, all codebases are incorrectly designed, but some are usable.

[^1]: Well, as much as an R package is a tool from scratch. You're usually standing on someone's shoulders, after all.
[^2]: I am going to set aside debates over whether this ought to be true, or if standards ought to be better. It's an important conversation, but the topics here are already too big for coverage in one small space. And besides, this is my fourth attempt at writing this blog post, and I never got near that before.
[^3]: It also highlights that, despite the assertion that code should be written to be read, reading code is a skill like reading any other sort of technical language. Nobody reads a scientific paper well their first time.
[^4]: I am deliberately ignoring the existence of friction shifting, because this whole thing is complicated enough already, so, posit indexed bar ends or indexed downtube shifters if it makes you feel more comfortable. Or Gevenalle's levers, if you'd rather. And then ignore that the front is usually friction even when the rear is indexed in any of those setups.
[^5]: Though some of these are evergreen.
[^6]: I'm looking at you, tidyverse.[^7] Though there are also fun (read: terrifying) incidents like [yanking left-pad](https://en.wikipedia.org/wiki/Npm_left-pad_incident).
[^7]: After more than a decade, it still changes as rapidly as if it were in beta. Not that I develop in the tidyverse, given a choice. Yes I'm a curmudgeon.
[^8]: Whether or not you're writing object-oriented code. Assuming that something is a tibble, or a list, is still an assumption about object structure, even if R is more about functions.
[^9]: Yeah, I know, triples lack that road je ne sais quoi. They're not cool. But they're effective.
[^10]: You could write a senior thesis on bike gearing, drivetrains, and derailleurs, and still not be done with the introduction. But suffice to say, a small desired change to one thing can cascade across the entire drivetrain. There's modularity, then there are ecosystems of dependent packages.
[^11]: Other than, of course, to see how far one can stretch a metaphor before it is well and truly beyond dead.
[^12]: It's a good question, and you really have no excuse.
[^13]: Some would say that it's so bike companies can sell you more things. They're probably not entirely wrong.
[^14]: Well, the nicer ones do, anyways. Some of them can only be downshifted in one.