---
layout: post
title: "Working effectively with iTerm2"
description: >
  A few handy settings tweaks and short keys to boost productivity in
  iTerm2.
date: 2012-03-22 22:42
comments: true
keywords: cli, iterm, iterm2, command line, Mac OSX, shortcuts, settings.
categories: 
- tools 
- tools-cli 

---

I have been using [iTerm](http://www.iterm2.com) in [daily work](http://www.favoritemedium.com) for almost a year now. 
Along the way, I learned a few handy **settings tweaks and shortcut keys to boost my productivity** in command-line environment. 

Install iTerm2
------
If you haven't heard of [iTerm](http://www.iterm2.com), it's a popular **open source alternative to Mac OS X Terminal**. 
Give it a try, download and install it from [http://www.iterm2.com](http://www.iterm2.com).

Fine-Tune Settings
-------
Launch iTerm, open **iTerm > Preferences** or just `Cmd + ,`.

### Open tab/pane with current working directory
Under **Profiles** tab, go to **General** subtab, set **Working Directory** to _"Reuse previous session's directory"_.

### Enable `Meta` key
To enable Meta key for [Bash readline editing](/blog/2012/01/04/shortcuts-to-move-faster-in-bash-command-line) e.g. `Alt + b` to move to previous word, under **Profiles** tab, go to **Keys** subtab, set **Left option key acts as:** to _"+Esc"_.

### Hotkey to toggle iTerm2
Under **Keys** tab, in **Hotkey** section, enable _"Show/hide iTerm2 with  a system-wide hotkey"_ and input your hotkey combination, e.g. I use `Ctrl + Shift + L`.

### Switch pane with mouse cursor
Under **Pointer**, in **Miscellaneous Settings** section, enable _"Focus follows mouse"_.


Handy Shortcut Keys
-------
Here's a set of **shortcut keys I commonly use**. You can always look for other shortcut keys in the iTerm menu.

### Tab navigation 
* open new tab `Cmd + t` 
* next tab `Cmd + Shift + ]`
* previous tab `Cmd + Shift + [`

### Pane navigation 
* split pane left-right `Cmd + d`
* split pane top-bottom `Cmd + Shift + d`
* next pane `Cmd + ]` 
* previous pane `Cmd + [` 

### Search
* open search bar `Cmd + f`
* find next `Cmd + g`

### Input to all panes
* input to all panes in current tab `Cmd + Alt + i`

### Clear screen
* clear buffer `Cmd + k`
* clear lines (Bash command) `Ctrl + l` 

### Zooming / Font Resize
* toggle maximize window `Cmd + Alt + =`
* toggle full screen `Cmd + Enter`
* make font larger `Cmd + +`
* make font smaller `Cmd + -`

iTerm lovers, did I miss anything out?

