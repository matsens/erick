---
layout: post
title: "Second-generation interactive shell tools"
image: goodnight-shell.png
description: "Modern replacements for classic interactive shell commands."
---

## Summary

* rather than naked shell, [try](#tmux) `tmux` for an organized collection of shells
* rather than local file system navigation, [try](#fzf) `fzf` to fuzzily get what you want
* rather than `cd`, [try](#autojump) `autojump`, which remembers your favorite locations
* rather than `find`, [try](#fd) `fd`, which has a much cleaner interface
* rather than `grep`, [try](#ag) `ag`, which is simpler and faster


## Introduction

The built-in shell environment is tremendously productive, but "second generation" tools can make it even more so.

If you want to try out these tools, you can run

    docker run -it quay.io/matsen/cozy-demo

to test them out in Docker.
There is some funkiness with the display on this Docker image (tmux pane splits use hashes and q's) but it's enough to get the idea.
See [the corresponding GitHub repository](https://github.com/matsen/cozy-demo) if you want to mimic the configuration.



# tmux

![]({{ "/public/images/goodnight-shell.png" | relative_url }})

When working on a modern desktop computer, it's easy to arrange multiple windows side by side, to switch between applications, etc.
On the command line this is achieved by use of a "terminal multiplexer".
This is especially lovely for working on remote machines, where one can detach and re-attach sessions with all their attendant windows.

The two most popular terminal multiplexers are [GNU Screen](https://www.gnu.org/software/screen/manual/screen.html) and [tmux](https://github.com/tmux/tmux/wiki).
This tutorial will demonstrate `tmux`, which is newer and more feature-rich.
We will also use our (non-default) configuration, which is vi-keybinding oriented (emacs-style keybindings are also available).

You send commands to these programs using a "prefix" key command followed by another keystroke.
In our configuration, the prefix is `Ctrl-a`, the most easily typed control combination.
For example, hitting `Ctrl-a`, letting go, then typing `c` opens a new terminal window.
We will abbreviate this combination `Ctrl-a c`.


### First steps in tmux

As a first step, I suggest just starting up tmux by executing `tmux` and trying out the following commands.
Open a few windows, type some commands, cycle through them, open a few panes in a single window, and move around through them.

* `Ctrl-a c` &nbsp; New window
* `Ctrl-a <Space>` &nbsp; Next window
* `Ctrl-a NUMBER` &nbsp; Move to window number `NUMBER`
* `Ctrl-a Ctrl-a` &nbsp; Move to the previously visited window
* `Ctrl-a v` &nbsp; Split window into panes vertically
* `Ctrl-a s` &nbsp; Split window into panes horizontally
* `Ctrl-a <arrow>` &nbsp; Move between panes
* `Ctrl-d` &nbsp; Close a pane or window (works outside of tmux)
* `Ctrl-a q` &nbsp; Kill a pane or window

Hopefully it's clear that the windows are complete environments which are indexed at the bottom, and the panes are sub-windows that have no associated indicator.
Each pane has a shell running inside of it.

The following commands are also essential:

* `Ctrl-a ?` &nbsp; Command help. Note that `/` will allow you to search.
* `Ctrl-a a` &nbsp; Actually send a `Ctrl-a` to the terminal (e.g. to move to the beginning of the line when editing a shell command)


### Detaching and attaching a session

Tmux can keep your window configuration and all of the running processes going, even if your terminal goes away.
This is called detaching and reattaching.

The polite way to detach and reattach is to (try this!)

* `Ctrl-a d` &nbsp; Detach
* `tmux attach` &nbsp; Reattach

Reattaching is also very handy if you are moving between laptops.
You can detach your remote session from one, log in on another, and then reattach on the second laptop.
Note that if you have different size terminal windows between the two machines the session can look wonky.
In that case simply use `tmux attach -d` which will redraw the session.

If your wifi drops you might just get disconnected to a remote machine, in which case you wouldn't have the opportunity to politely detach.
*That's fine.*
In that case, just re-connect to your remote machine, and `tmux attach`.


Also note that `tmux attach` will fail if you don't have a session open already; if that happens just enter `tmux` to start a new session.


### Resizing panes

* `Ctrl-a <` &nbsp; Move vertical split left
* `Ctrl-a >` &nbsp; Move vertical split right
* `Ctrl-a +` &nbsp; Move horizontal split up
* `Ctrl-a =` &nbsp; Move horizontal split down

Note: You can hit `Ctrl-a` once, and press the resize operator a number of times (or hold it) to do larger resizes.
For example, try `Ctrl-a <<<<`.


### Naming & finding windows

* `Ctrl-a ,` &nbsp; Name a window
* `Ctrl-a '` &nbsp; Present a list of windows (by name), which you can choose from by `<arrow>` and `Enter` to switch to a window
* `Ctrl-a <numeric>` &nbsp; Switch to a window by number.


### Scrolling and copy/paste in tmux

When you have lots of output, it's nice to be able to scroll up and down through history.
Pressing `Ctrl-a [` will place you in scroll mode.
Use can now use arrow keys, PageUp/PageDown keys, or `Ctrl-u`/`Ctrl-d` to scroll through the history.
You can search up through history with `?` and down with `/`.

From this mode, you can also press `Space` to enter copy-mode, arrow keys to specify a selection range, and `Enter` to copy the selection.
To paste the selection, use `Ctrl-]`.

Note: if you are running tmux 2.1 or greater (check with `tmux -V`) and you really like your mouse scroll wheel, you can try `set -g mouse on` which will drop you into tmux scroll mode when you use the scroll wheel.
But I suggest keeping your fingers on the keyboard and ignoring the "rat".


### Advanced tmux tips

You can also use `h`, `j`, `k`, and `l` in place of the arrow keys, as in `vim`.

If you want to pipe to the tmux copy/paste buffer, try this alias:

    alias tc='tmux loadb -'

With this alias I can, for example, `ls /very/long/path.txt | tc` and then paste rather than having to do an explicit select and copy.


There are many more commands than we have connected with command sequences.
See the tmux man page for a complete list.
`Ctrl-a :` will put you at a prompt that will allow you to execute an arbitrary command.


### Using tmux with ssh

Ssh keys in a long-running tmux session require some care.
I use [keychain](https://www.funtoo.org/Keychain) to automate `ssh-agent` and friends together with tmux like so:

    eval `keychain --eval id_rsa` && tmux attach -d

Sometimes it seems that ssh connections inside those sessions need a little refresh.
For that I use [this shell command](https://gist.github.com/matsen/b121999f861cd001c6814a20d032fa53).


### You can get fancy with tmux

Some people love complex tmux setups with tons of sub-panes with things running in them.
I prefer to have a simple setup where things can get set up and torn down easily.
If you like a fancy setup, you can use something like [tmux-resurrect](https://github.com/tmux-plugins/tmux-resurrect) to maintain such setups on your local machine between reboots.
There are also tmux plugin managers, etc, but just a simple configuration file is all I've needed.

You can also have lots of different detached sessions, give them names, etc.
I used to do this, with each session corresponding to a single project.
For me it ended up being too much work to manage, but perhaps it fits for you.


# fzf

We all love tab completion for matching files.
But sometimes it too can be painful.
Consider this list of files, which is some the various posts that have appeared on [fredhutch.io](http://fredhutch.io):

```
2014-04-24-command-line.md        2015-04-06-R-sequence-analysis.md
2014-04-27-terminal-multiplex.md  2015-04-23-ucsc-xena-workshop.md
2014-05-09-galaxy.md              2015-04-24-third-galaxy-101.md
2014-05-11-editing.md             2015-05-04-spring-2015-r.md
2014-05-17-git.md                 2015-05-20-spring-2015-unix-hpc.md
2014-05-20-R.md                   2015-06-03-summer-bioinfo.md
2014-07-13-toolbox.md             2015-08-26-data-center-tour.md
2014-07-16-aws.md                 2015-08-27-cloud-computing.md
2014-08-13-synapse.md             2015-12-02-galaxy-rna-seq.md
2014-09-19-R-course.md            2016-01-21-cds-git.md
2014-10-20-introductory-r-.md     2016-02-25-immunespace.md
2014-10-22-shippable.md           2016-03-01-i-heart-pandas.md
2014-10-27-bioconductor.md        2016-03-28-gizmo-brownbag.md
2014-11-03-labkey.md              2016-06-14-scicomp-unix-hpc.md
2014-11-07-intermediate-R.md      2016-07-28-galaxy-101.md
2014-12-09-hidra.md               2016-08-23-chipseq-class.md
2014-12-15-scicomp-unix-hpc.md    2016-08-31-rnaseq-class.md
2015-02-11-scicomp-unix-hpc.md    2016-09-08-inkscape.md
2015-03-09-introductory-R.md      2016-09-27-intro-r.md
2015-03-12-rollout-galaxy-101.md  2016-10-03-introbio.md
2015-04-02-april-galaxy-101.md    2017-01-05-command-line-cozy.md
```

to tab-complete through these files is a pain, because I actually want to choose a file by the text portion of the descriptor, which is past the date.

What we really want is _fuzzy finding_, which allows us to match via partial substring matches.
By this I mean that arbitrary substrings of your query string get matched with arbitrary substrings of your target strings, in an ordered fashion (see below for an example).
This is available [as part of the GitHub web interface](https://github.com/blog/793-introducing-the-file-finder) for finding files, and actually even in the Chrome address bar.

[fzf](https://github.com/junegunn/fzf) brings fuzzy finding to the shell.
For example, if I want the posts from 2015 that contain `galaxy`, I could type `2015galaxy`, and 💥:

```
  2015-03-12-rollout-galaxy-101.md
  2015-04-24-third-galaxy-101.md
  2015-04-02-april-galaxy-101.md
  2014-05-09-galaxy.md
> 2015-12-02-galaxy-rna-seq.md
  5/41
> 2015galaxy
```

we get a list of the files that contain `201` `5` and `galaxy`.
In fact, I could have typed `15axy` and gotten the same result, because it matches the same set of files.
I can then type more characters to limit the search, or use the arrow keys to move between results, and then return to select.
Note that there is a "false positive" here, namely `2014-05-09-galaxy.md` which is from 2014 but has a 5 in the month.
That's just a natural consequence of the way fuzzy finding works.

Here are two very easy ways to get started with fzf from the shell:

* `Ctrl-t` drops you into the fzf fuzzy command completer from the shell prompt. For example, `less Ctrl-t` will allow you to find the file that you want to `less`.
* `Ctrl-r`, the usual reverse search on your history, gets replaced by the fuzzified version. This is great and allows you to be vague about what you're looking for.

For many more uses, including vim integration, see the [fzf](https://github.com/junegunn/fzf) GitHub page.


# autojump

[autojump](https://github.com/wting/autojump) is a `cd` replacement that learns.
It keeps a record of where you have been in the past, and given multiple directories that match an argument, chooses the one that you have been in the most.
The autojump command is `j`.

For example, I frequently visit our data directory on the shared filesystem at `/fh/fast/matsen_e/data`.
Instead of typing out that full filename I can just type `j data` to `cd` there.

This works with partial filenames as well.
In the example Docker container I have a few directories that have cats in them, and `j cats` will take you to `/root/cats/siamese-mostpopular`, which is the most popular subdirectory containing `cats`.

If you are in the most popular directory with a certain string, then running the same `j` command will take you to the second most popular.
For example, repeating `j cats` a second time would take us to `/root/cats/bengal`, which is the subdirectory we've visited the second most.
(We could have used `cat` here as the argument to `j` instead of `cats` but I didn't want to confuse things with the `cat` shell command.)

The information about what directories you spend the most time in is stored in `~/.local/share/autojump/autojump.txt`.
For our example Docker image it looks like this:

    50.0	/root/cats/siamese-mostpopular
    30.0	/root/cats/siamese-lesspopular
    40.0	/root/cats/bengal
    10.0	/root/work

The left-hand column here is the popularity index and the right hand is the directory.


# fd

[GNU Find](https://www.gnu.org/software/findutils/manual/html_mono/find.html) is incredibly powerful, but has an annoying interface.
The `fd` command fixes that.

* `fd PATTERN` finds a file in the current directory whose filename matches `PATTERN`
* `fd PATTERN -x COMMAND` executes `COMMAND` for each file matching `PATTERN`

For example, `fd .txt -x head -n 1` will perform `head -n 1` on each file matching `.txt`.

The `fd` command has sensible defaults for the modern environment, such as ignoring `.git` directories and files that appear in `.gitignore`.
See the [fd GitHub page](https://github.com/sharkdp/fd) for more examples and detail.


# ag

`ag` is a faster replacement for `grep` that has a nicer input and output interface.

For example, as I'm writing this I might look for instances of the word "fancy" in the repository for this blog.
Using `ag fancy` gets me:

    _posts/2019-11-05-travis.md
    22:It can do all sorts of fancy pipelines, but here we'll just get it to do one simple thing: run a command in a Docker container.

    _posts/2019-12-31-command-line-cozy.md
    145:### You can get fancy with tmux
    149:If you like a fancy setup, you can use something like [tmux-resurrect](https://github.com/tmux-plugins/tmux-resurrect) to maintain such setups on your local machine between reboots.

which shows clearly that this word appeared once in a post about Travis, and twice in this file (before I wrote this section).

In contrast, we can try the same thing using `grep -r fancy`

    _posts/2019-11-05-travis.md:It can do all sorts of fancy pipelines, but here we'll just get it to do one simple thing: run a command in a Docker container.
    _posts/2019-12-31-command-line-cozy.md:### You can get fancy with tmux
    _posts/2019-12-31-command-line-cozy.md:If you like a fancy setup, you can use something like [tmux-resurrect](https://github.com/tmux-plugins/tmux-resurrect) to maintain such setups on your local machine between reboots.
    _site/feed.xml:&lt;h3 id=&quot;you-can-get-fancy-with-tmux&quot;&gt;You can get fancy with tmux&lt;/h3&gt;
    _site/feed.xml:If you like a fancy setup, you can use something like &lt;a href=&quot;https://github.com/tmux-plugins/tmux-resurrect&quot;&gt;tmux-resurrect&lt;/a&gt; to maintain such setups on your local machine between reboots.
    _site/feed.xml:It can do all sorts of fancy pipelines, but here we’ll just get it to do one simple thing: run a command in a Docker container.&lt;/p&gt;
    _site/2019/11/05/travis.html:It can do all sorts of fancy pipelines, but here we’ll just get it to do one simple thing: run a command in a Docker container.</p>
    _site/articles/2019/12/31/command-line-cozy.html:<h3 id="you-can-get-fancy-with-tmux">You can get fancy with tmux</h3>
    _site/articles/2019/12/31/command-line-cozy.html:If you like a fancy setup, you can use something like <a href="https://github.com/tmux-plugins/tmux-resurrect">tmux-resurrect</a> to maintain such setups on your local machine between reboots.

First, things aren't as nicely organized into sections.
Second, grep looks into all of the files, including the ones that are in `.gitignore`.
The git-ignored files in `_site` are auto-generated and they only add noise to my search.

`ag` is also very fast and has lots of nice editor integrations.
See the [ag GitHub page](https://github.com/ggreer/the_silver_searcher) for more details.


# Conclusion and future directions

We've looked at a few tools that can help make it easier to interact with the shell.

We haven't considered the shell itself.
If you spend a lot of time in the shell, you might consider [zsh](https://en.wikipedia.org/wiki/Z_shell) or [fish](https://fishshell.com/).

Or you could just be happy with the defaults.
Every minute you spend tweaking your configuration is a minute you aren't doing creative and beautiful things.

---

Thank you to [Chris Small](https://github.com/metasoarous) and [Connor McCoy](https://github.com/cmccoy), two previous group members who pointed me to some of these tools.
Chris wrote parts of the section on tmux as part of his course material for [fredhutch.io](http://fredhutch.io).
