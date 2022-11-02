---
title: Hello Suse!
date: 2022-11-02 17:00:00
categories: [Linux]
tags: [linux, opensuse, tumbleweed, debian, ubuntu]
---

I have never been much of a distro-hopper. For the past 10 years or so I have
pretty much stuck to Debian or Ubuntu. Debian testing has always been my
operating system of choice for personal use and Ubuntu for work. In as much
as snap on Ubuntu has its problems, it is quite convinient when it comes to
finding and installing a number of packages that I use at work (slack, vscode,
and a bunch of other stuff).

## Despair at the foot of mount Redmond

About 1 year ago I started a job at a place that's mostly a Windows house.
I was kind of forced to use Windows because of some stupid software I was
supposed to use at work that's Windows only (SSIS). So I had Windows on
my work computer with WSL set up to meet all my needs outside of
visual studo + SSIS. I honestly tried to force myself to use Windows but
overtime it was becoming clear to me that Windows isn't home. From the GUI
to issues with WSL itself I was pushed well over the edge. It was the little
things, switching between Windows of the same application (`ALT-`` in Gnome)
is not exactly available, the task switcher being a bit jarring
(not as smooth as what you get in Gnome or KDE), half-assed workspaces and
window overview, docker builds failing because WSL does some funny shit with
domain name resolution, quickly running out of memory, windows breaking itself
for reasons I couldn't explain, and a billion other issues. All these issues
combined with the fact that we had started transitioning away from SSIS
to slightly more sensible and OS agnostic tools at work made me jump ship.
Going back to an environment I was much more comfortable with was now a
simple and straightforward move.

## This no longer feels like the valley of plenty promised in the canon

I grabbed me a copy of Ubuntu 22.04 and installed it on my computer.
Left Windows intact, in case I need to work on something that
required this shitty OS (fortunately for me, I have not needed to
do this at all). With Ubuntu installed, one would think things would
become smoother. Of course they were up to a certain extent, however
my overall experience with Ubuntu 22.04 did leave a foul taste in my
mouth. I can say without a shadow of doubt that Ubuntu 22.04 is the
one of the worst releases of Ubuntu in the past 10 years. Just to
name some of the things of I faced: systemd's OOM daemon that silently
murders processes when memory is running out, lock screen freezing
randomly, unable to share screen in certain applications (mostly
those installed through snap - could fix them but the problems
would magically come back after an update or something). About a
month or two ago I told myself that I have seen enough, I can't live
like that, I am going away from Ubuntu. Now, at that point I could
have gone back straight to Debian but something pushed me towards trying
out something new. I had three options I was looking at: Fedora,
Arch, and Opensuse Tumbleweed. Long story short I picked Suse Tumbleweed and
I must say, it's one of the best decisions I have ever made in life.

## Found peace in Suseland

To be continued...
