---
title: Terminal Markdown Viewer and MarkDown SpellChecker
date: 2017-09-07 02:00:16
tags:
  - Markdown
  - Tools
  - Utils
  - Linux
---

It's been a while that i've updated this posts. So let me start for something i've found out the other day.

## Terminal Markdown Viewer

I though it would be cool to have some tool that could parse me Markdown files and present them nicely formatted, like `cat` command.

And it seems i'm not the only one :)

https://github.com/axiros/terminal_markdown_viewer

In order to install just follow the instruction on the site. Pip worked well for me:

```
pip install mdv
```

I've updated my PATH environment var on `.bashrc` with:

```
export PYTHONPATH="${PYTHONPATH}:/${HOME}/.local/lib/python2.7/site-packages"
export PATH="$PATH:~/.local/bin"
```

I can now execute `mdv` command try it out it some .md files :D

## A Markdown spell checking tool.

The other thing is regarding spell-check, as you can probably notice i really do a lot of errors, but i don't wont to use a high resource editor just because of this how about a MD spell checking tool ?

Well i'm trying this one ad i'm using [hexo][http://hexo.io] for the page generation and it uses already npm modules just added a new one 

### install

```
npm i markdown-spellcheck -g
```

One can run interactive like

```
mdspell "**/*.md"
```

It's a bit slow but the database would grow in terms, i'm giving a try to it.

I also add to add `.spelling` no to keep the dictionary database on git. I probably would add it later.

Cheers,
RR