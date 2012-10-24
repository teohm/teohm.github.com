---
layout: post
title: "Shortcuts to move faster in Bash command line"
description: >
  I stopped using arrow keys recently, and start using shortcut keys to 
  move around and edit efficiently on the Bash command line.
date: 2012-01-04 20:13
comments: true
keywords: bash, command line editing, shortcuts, arrow keys, tips.
categories: 
- command-line
- bash
---

Nowadays, I spend more time in Bash shell, typing longer commands. One of my new year resolutions for this year 
is to **stop using left/right arrow keys to move around in the command line**. 
I learned [a few shortcuts](http://pivotallabs.com/users/hunter/blog/articles/1925-terminal-beyond-ctrl-a-and-ctrl-e)
a while ago. 

Last night, I spent some time to read about **"Command Line Editing" in [the bash manual](http://www.gnu.org/software/bash/manual/bashref.html#Command-Line-Editing)**.
The bash manual is a well-written piece of documentation. I think I should read it more often. 

Well, here's the new shortcuts I learned:

Basic moves
----------
* Move back one character. `Ctrl` + `b`
* Move forward one character. `Ctrl` + `f`
* Delete current character. `Ctrl` + `d`
* Delete previous character. `Backspace`
* Undo. `Ctrl` + `-`

Moving faster
-------------
* Move to the start of line. `Ctrl` + `a`
* Move to the end of line. `Ctrl` + `e`
* Move forward a word. `Meta` + `f` *(a word contains alphabets and digits, no symbols)*
* Move backward a word. `Meta` + `b`
* Clear the screen. `Ctrl` + `l`

**What is Meta?** `Meta` is your `Alt` key, normally. For **Mac OSX user, you need to 
enable it yourself**. Open *Terminal > Preferences > Settings > Keyboard*,
and enable *Use option as meta key*. `Meta` key, by convention, is used for operations on word.

Cut and paste ('Kill and yank' for old schoolers)
-------------
* Cut from cursor to the end of line. `Ctrl` + `k`
* Cut from cursor to the end of word. `Meta` + `d`
* Cut from cursor to the start of word. `Meta` + `Backspace`
* Cut from cursor to previous whitespace. `Ctrl` + `w`
* Paste the last cut text. `Ctrl` + `y`
* Loop through and paste previously cut text. `Meta` + `y` (use it after `Ctrl` + `y`)
* Loop through and paste the last argument of previous commands. `Meta` + `.`

Search the command history
--------------
* Search as you type. `Ctrl` + `r` and type the search term; Repeat `Ctrl` + `r` to loop through results.
* Search the last remembered search term. `Ctrl` + `r` twice.
* End the search at current history entry. `Ctrl` + `j`
* Cancel the search and restore original line. `Ctrl` + `g`

Need more?
-----
- A comprehensive [**bash editing mode cheatsheet**](http://www.catonmat.net/blog/bash-emacs-editing-mode-cheat-sheet/) by Peteris Krumin ([catonmat.net](http://catonmat.net)).
- **Vim users!** Do you know you can switch to Vi-style editing mode?
  Here: [vi-style cheatsheet](http://www.catonmat.net/blog/bash-vi-editing-mode-cheat-sheet/).
- Bash command line editing is actually handled by **GNU Readline Library**. So 
  just dive into [Readline manual](http://www.gnu.org/software/readline/#Documentation) 
  for everything else.
