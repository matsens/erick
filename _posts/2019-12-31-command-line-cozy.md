---
layout: post
title: "Second-generation shell interaction"
category: articles
---

## Summary

* rather than naked shell, [try](#tmux) `tmux`
* rather than local directory navigation, [try](#fzf) `fzf`
* rather than `cd`, try `autojump`
* rather than `find`, [try](#fd) `fd`
* rather than `grep`, try `ag`


## Introduction

The default shell environment is tremendously productive, but "second generation" tools can make it even more so.

If you want to try out these tools, you can run

    docker run -t quay.io/matsen/cozy-demo

to test them out in Docker.
There is some funkiness with this Docker image (the pane splits use hashes and q's to divide the pane) but it's enough to get the idea.
See [the corresponding GitHub repository](https://github.com/matsen/cozy-demo) if you like the configuration and want to use it for your setup.



## tmux

When working on a modern desktop computer, it's easy to arrange multiple windows side by side, to switch between applications, etc.
On the command line this is achieved by use of a "terminal multiplexer".
This is absolutely essential for working on remote machines, where one can detach and re-attach sessions with all their attendant windows.

The two most popular terminal multiplexers are [GNU Screen](https://www.gnu.org/software/screen/manual/screen.html) and [tmux](https://github.com/tmux/tmux/wiki).
This tutorial will demonstrate `tmux`, which is newer and more feature-rich.
We will also use our (non-default) configuration.

You send commands to these programs using a "prefix" key command followed by another keystroke.
In our configuration, the prefix is `Ctrl-a`, the most easily typed control combination.
For example, hitting `Ctrl-a`, letting go, then typing `c` opens a new terminal window.
We will abbreviate this combination `Ctrl-a c`.


### First steps in tmux

As a first step, I suggest just starting up tmux by executing `tmux` and trying out the following commands.
Open a few windows, type some commands, cycle through them, open a few panes in a single window, and move around through them.

* `Ctrl-a c` - New window
* `Ctrl-a <Space>` - Next window
* `Ctrl-a v` - Split window into panes vertically
* `Ctrl-a s` - Split window into panes horizontally
* `Ctrl-a <arrow>` - Move between panes
* `Ctrl-d` - Close a pane or window
* `Ctrl-a q` - Kill a pane or window

Hopefully it's clear that the windows are complete environments indicated at the bottom, and the panes are sub-windows that have no associated indicator.
Each one of these has a shell running inside of it.

The following commands are also essential:

* `Ctrl-a ?` - Command help. Note that `/` will allow you to search.
* `Ctrl-a a` - Actually send a `Ctrl-a` to the terminal (e.g. to move to the beginning of the line when editing a shell command)


### Detaching and attaching a session

Tmux can keep your window configuration and all of the running processes going, even if your terminal goes away.
This is called detaching and reattaching.

The polite way to detach and reattach is to (try this!)

* `Ctrl-a d` to detach
* `tmux attach` to reattach

However, if your wifi drops you might just get disconnected to a remote machine, in which case you wouldn't have the opportunity to politely detach.
*That's fine.*
In that case, just re-connect to your remote machine, and `tmux attach`.

This is also very handy if you are moving between laptops.
You can detach your remote session from one, log in on another, and then reattach on the second laptop.
Note that if you have different size terminal windows between the two machines the session can look wonky.
In that case simply use `tmux attach -d` which will redraw the session.

Also note that `tmux attach` will fail if you don't have a session open already; if that happens just enter `tmux` to start a new session.


### Resizing panes

* `Ctrl-a <` - move vertical split left
* `Ctrl-a >` - move vertical split right
* `Ctrl-a +` - move horizontal split up
* `Ctrl-a =` - move horizontal split down

Note: You can click `Ctrl-a` once, and press the second key a number of times (or hold it) to do larger resizes


### Naming & finding windows

* `Ctrl-a ,` will let you name a window
* `Ctrl-a '` presents a list of windows (by name)
  * `<arrow>` and `Enter` to switch to a window
* `Ctrl-a <numeric>` switches to a window by number.


### Scrolling and copy/paste in tmux

When a noisy program floods a tmux pane, your mouse wheel won't let you scroll, like in a normal shell session.
Pressing `Ctrl-a [` will place you in scroll mode.
Use can now use arrow keys or `Ctrl-u`/`Ctrl-d` to scroll through the history, and search with `/`.

From this mode, you can also press `Space` to enter copy-mode, `<arrow>` keys to specify a collection, and `Enter` to copy the selection.
To paste the selection, use `Ctrl-]`.


### Advanced tmux tips

You can also use `h`, `j`, `k`, and `l` in place of the arrow keys, as in `vim`.

If you want to pipe to the tmux buffer, try this alias:

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


## fzf

We all love tab completion.
But sometimes it too can be painful.
Consider this list of files, which is the various posts that have appeared on fredhutch.io:

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

Thus we can use _fuzzy finding_, which allows us to use arbitrary substrings to find what I want.
This is available [in GitHub](https://github.com/blog/793-introducing-the-file-finder) for finding files, and actually even in the Chrome address bar.
With fuzzy finding, you can just type substrings of your desired string and the matcher will find items that contain those substrings.

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

we get a list of the files that contain `2015` and `galaxy`.
In fact, I could have typed `15axy` and gotten the same result, because it matches the same set of files.

Here are two very easy ways to get started with fzf:

* `Ctrl-t` drops you into the fzf fuzzy command completer from the shell prompt. For example, `less Ctrl-t` will allow you to find the file that you want to `less`.
* `Ctrl-r` does reverse search on your history

For many more uses, including vim integration, see the [fzf](https://github.com/junegunn/fzf) GitHub page.


## autojump

[autojump](https://github.com/wting/autojump) is a `cd` replacement that learns.
It keeps a record of where you have been in the past, and given multiple options, chooses the one that you have been in the most.
The autojump command is `j`.

For example, I frequently visit our data directory on the shared filesystem at `/fh/fast/matsen_e/data`.
Instead of typing out that full filename I can just type `j data`.

This works with partial filenames as well.
In the example Docker container I have a few directories that have cats in them, and

    j cat

will take you to `/root/cats/siamese-mostpopular`, which is the most popular subdirectory containing `cat`.

If you are in the most popular directory with a certain string, then running the same `j` command will take you to the second most popular.
For example, repeating `j cat` a second time would take us to `/root/cats/bengal`.


## fd

[GNU Find](https://www.gnu.org/software/findutils/manual/html_mono/find.html) is incredibly powerful, but has an annoying interface.
The `fd` command fixes that.

* `fd PATTERN` finds a file in the current directory whose filename matches `PATTERN`
* `fd PATTERN -x COMMAND` executes `COMMAND` for each file matching `PATTERN`

For example, `fd .txt -x head -n 1` will perform `head -n 1` on each file matching `.txt`.

The `fd` command has sensible defaults for the modern environment, such as ignoring `.git` directories and files that appear in `.gitignore`.
See the [fd GitHub page](https://github.com/sharkdp/fd) for more examples and detail.


## grep & friends

Moving on to finding files by their content, the first step is the classic `grep`.

To find occurrences of the string "smooshable" in any file contained in the current directory, just write

```
grep -R smooshable
```

where the `-R` is for recursive search across the directory tree.

I don't actually use recursive grep much.
If I'm looking for something in a git repository I use `git grep`:

```
git grep smooshable
```

which is fast and tidier because it doesn't find things in the `.git` directory, etc.

If I have a lot of big files to search through, [ag, the silver searcher](https://github.com/ggreer/the_silver_searcher) is a great tool.
It searches recursively by default, so in this example

```
ag smooshable
```

will get you a nicely formatted list of instances.
Note that ag also has lots of nice editor integrations.




## History

Your "history" is the list of commands you have entered.
The easiest way to get back in your history is just to hit the up arrow.
Repeatedly hitting up arrow will take you back in time, while down arrow takes you forward in time.
When we see a command that we like, we can hit `Return` to execute that command, or use the left and right arrow keys to move to a place where we can edit it.

Here are some commands to help you browse that history:

* `history`: gives the full history; likely to be too much too be useful
* `history | tail`: the history truncated to the most recent commands
* `history | less`: history made more navigable

You can also search through your history.
Hitting `Ctrl-r` brings up reverse interactive search.
In this example, I typed `tags`, which brings up the most recent command containing the string `tags`:

```
$ git describe --tags --long
bck-i-search: tags_
```

You can cycle through earlier commands by hitting `Ctrl-r` again.
If you want to cancel your reverse search, use `Ctrl-g`.

It appears that OS X truncates your history at a measly 500 lines.
Phooey on that!
Put this in your `.bashrc` to get an unlimited history:

```
export HISTFILESIZE=
export HISTSIZE=
```

More about your `.bashrc` below.
Note that [this isn't a perfect solution and a better one exists](http://superuser.com/a/664061), but it's good enough.

### The last word in history

There is a very handy trick to go cycle through the last words in your history.
Say I'm doing the following:

```
git add horribly/long/path/to/file.txt
git status
```

and now I remember I wanted to make one more modification to `horribly/long/path/to/file.txt`.
One can hit the up arrow two times and then replace `git add` with an invocation of an editor.

But there's something slicker.
`Alt-.` cycles through the previous last words in the command history and puts them on the command line.
In this example, I could say `vi ` `Alt-.` `Alt-.` and `vi horribly/long/path/to/file.txt` would result.

For the OS X fans out there, you will use `Option` in place of `Alt`, which may require [some configuration](http://osxdaily.com/2013/02/01/use-option-as-meta-key-in-mac-os-x-terminal/) (note `Meta` is another name for `Alt` in this context).




## Interacting with the web

To get something off the web, use `wget` and then the web address.
This is handy in combination with the "raw" address for files on GitHub (available as a button on the right hand side of a file's page), e.g.:

```
wget https://raw.githubusercontent.com/git/git/master/contrib/completion/git-prompt.sh
```

Another related tool is [curl](https://curl.haxx.se/docs/manual.html), which is quite powerful.

If you actually have to interact with the web via the command line, you can always use `lynx`.
If you want to watch nyan cat, you can do so by executing

```
telnet nyancat.dakko.us
```


## Customization

If you want to customize the way your shell works, you need to run certain programs each time your shell starts.
If you use bash (and you probably use bash if you don't know what shell you use) then you can run these programs by putting them in your `.bashrc`.
Perhaps you already know about this file for modifying your `PATH` variable.

For example, here are a few of my favorite aliases:

```
alias lst='ls -clth | head'
alias ll='ls -lh'
alias path='readlink -f'
```

If I want those to be there every time I log in, I want to put those lines in my `.bashrc`.


### Git prompt

I don't mind what shells or editors people in my group use, but I really feel strongly that everyone should use a shell prompt that displays information about git status.
Not having this inevitably leads to confusion with partial commits.

The git folks understand this and have made a [git prompt script](https://github.com/git/git/blob/master/contrib/completion/git-prompt.sh).
Bash folks: you can grab this using the wget command above, move it to `~/.git-prompt.sh`, and throw this in your `.bashrc`:

```
source ~/.git-prompt.sh
export GIT_PS1_SHOWDIRTYSTATE=1
export GIT_PS1_SHOWUNTRACKEDFILES=1
PS1='[\u@\h \W$(__git_ps1 " (%s)")]\$ '
```

If you use zsh you probably have all of this configured, but I'd suggest trying out [antigen](https://github.com/zsh-users/antigen) which has a plugin system making all this trivial.


## Moving around

We all know and love `cd`, but moving around with raw `cd` can get tedious.
First a few little tricks:

* `cd`: moves you back to your home directory
* `cd -`: moves you back to your previous location

But still, getting deep in a directory tree takes a lot of typing and/or tab completion.
For this reason, I use [autojump](https://github.com/wting/autojump), which remembers where you've been so you can move around more quickly.
Before you install that tool, though, take a look at the next section, which also offers a faster way to navigate deep in directory trees.




If you aren't yet familiar with the shell, try the two-session [shell tutorial](https://github.com/fredhutchio/tfcb_2019/tree/master/lectures/lecture09) I taught as part of a class for Fred Hutch for UW MCB students.


