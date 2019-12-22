---
layout: post
title "Seriously, am I in vim mode or not"
date: 2019-11-03
--- 

## I use vim
I use vim on a daily basis. I edit code with it, write these blogs posts, rename
files with ranger etc etc. I like vim and have gotten pretty use to using it,
especially for editing code. I am not claiming to be a vim ninja who edits at
the speed of light or anything you understand, and I certainly use my fair share
of plugins ([YouCompleteMe](https://github.com/ycm-core/YouCompleteMe) by Valloric is a godsend). But on the whole, I mostly
use vim for vim. I use a lot of the builtin functionality in my workflow. 

## I also use IDEs
I also use IDEs sometimes, as sometimes they are just the best tool for the job. 
As much as I love the efficiency of vim for text editing, it is not always the
easiest way to manage large projects. Sure, I know my around the terminal and I
can edit my .vimrc on the fly. But when I have a project that has a GUI section
and business logic spread out over 50+ files, most of which need to be edited
simultaneously, window splits in vim are not great, and adding a mapping to
compiler and run everything does not always work. Sometimes, it is just better
to have easily manageable run configurations and built in debugging features with
out the hassle.

## Using both does not work
So what is the issue? Well the issue is that some IDE producers want to help
me out by providing me with a vim, or there is a vim mode
plugin available. This however does not really help most of the time.
For example, the vim emulation plugin for Intelli-j. Really all you get is the
vim key navigation and a few of the modes. But it does not read my .vimrc, so
all my customization (the main reason I use vim) is not there. Neither are some
of the simple features (like gg=G) or vim's buffer search and replace. It also
overwrites some of the normal vim commands with its on hotkeys and does not
really say which. Which leads to the nightmare of trying to figure what works
and what doesn't. 

Vim mode can be toggled, but the command to do it is <ctrl>+<alt>+C which is the
same as terminal copy. Which let to me thinking I was in vim mode when I really
wasn't, or vice versa. I finally just disabled the feature entirely because it
was so much of a nuisance.

## The moral of the story
Just use tools the way they were intended, vim is great for some things, but the
integrated nature of IDEs is sometimes necessary. Vim's design sometimes
contradicts the goals of IDEs so it leads to a lot of unnecessary pain to mix
them.  
