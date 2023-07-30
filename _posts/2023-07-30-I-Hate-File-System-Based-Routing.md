---
title: I Hate File System Based Routing
date: 2023-07-30 22:00:00
categories: [Programming]
tags: [programming, javascript, sveltekit]
---

I have been using Sveltekit on a number of projects for a while now. Whilst I like some
of its ideas, there is one thing that just undoes the whole thing for me. File system
based routing, it sounds like a good idea but in practice it's annoying as f%$k on any
moderately complex application. It pretty much breaks how I work with any of my text
editors of choice (neovim / vscode).

First problem, tabs are rendered less useful. When you have 10 files open that are all
named `+page.svelte` in tabs, it becomes harder to know which tab to go to for a
specific file. Yes, some text editors include some information about the directory
containing the file but it's still a long way from improving the user experience.
In the absence of this file system routing nonsense, finding the file with the product
edit logic is as simple as just scanning for for a tab with a label like
`EditProduct.svelte`, not trying to make sense of labels that look something
like `+page.svelte (../[id])`.

Secondly and the biggest deal breaker for me, it breaks fuzzy file search. Jumping between
files in a text editor for me involves using a fuzzy file search tool like FZF or vscode's
*<Ctrl-P>* thingy. I type the name of the file I want and hit enter to jump to it.
I find that workflow super fast, I have been using it for years. I don't need to reach for
a mouse and start clicking through menus and what not. No need to click through some file
system tree view to search for some file. With file system based routing, fuzzy finding
files becomes 10 times harder. Searches become less precise since you mostly match
directories containing a shit ton of files and directories. Fuzzy finding files becomes
search for what you want then scroll through a list containing a bunch of files you are
not interested in. I am having to unlearn how I use a text editor for the sake of
Sveltekit.

A work around for these problems involves pushing as much logic out of Sveltekit's
routes folder to a component hierarchy outside. Basically, your `+page.svelte` just
imports a component from elsewhere containing all the logic. The component's file can
have a nice name that's easy to remember (e.g. `pages/EditProduct.svelte`). However, by
design Sveltekit pulls you to have a good amount of pages' logic in your routes directory.
The logic needed to do stuff in your component is normally spread out in a number of
files (`+page.svelte`, `+page.js`, `+page.server.js`, `+layout.server.js`, etc).
Ideally, you don't want your logic spread all over so naturally you gravite towards having
a big part of pages' logic in `+page.svelte`. Frameworks are supposed to make work easier.
In this case, I am needing to find work arounds in order to make working with the
framework easier. In my opinion, this is a sure sign that something is awfully wrong.

