---
title: "Environment Variables Rule Everything Around You"
date: 2018-06-23T00:18:42-04:00
categories:
 - "software"
 - "nix"
tags:
 - "bash"
 - "zsh"
 - "csh"
 - "fish"
---

There's lots of discussion for operating systems about processes, ipc, files and such, I wanted to take a moment and touch on some internals I find
incredibly useful in my day to day work. Environment variables!

NOTE: I'm really only covering Linux/OS X/POSIX class systems here, Windows (and PowerShell) have some slightly different edges.

# What
Via [Wikipedia](https://en.wikipedia.org/wiki/Environment_variable) - an environment variable is
"a dynamic-named value that can affect the way running processes will behave on a computer."

That's really, all there is to it. The [POSIX.1 Specification](http://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap08.html) provides
conventions for application writers, but if you want to make a program that behaves differently there's nothing stopping you! As part of the C standard,
most languages will provide a `getenv()` function - it is often part of the `os` library. The only other guidance I have is environment variables are
_usually_ upper case for the key, but some operating systems will allow you to set lower-cased names. Said operating systems are also generally case
sensitive, so setting up `path` is not the same as setting up `PATH`.

## A Diversion - shell variables, ssh, docker and sudo

For people interacting with computers via some sort of command line, that command line is often a general purpose shell. This brings up a whole new layer
of confusion, as your typical shell will commingle environment variables _and_ variables internal to your shell session. The key things to keep in mind are:

- shell variables are not passed along to new processes started by the shell
- most shells can promote a shell variable to an environment variable via the `export` builtin
- `ssh`, since it typically starts a new user login against a remote host, does not bring along environment variables without specific configuration.
- `docker` is similar to the `ssh` case because the underlying machinery starts a new environment image.
- `sudo` is an interesting case. While it is a child of your process tree, as a security control, it explicitly *resets* the environment in most cases.

### Can I just list all the shell variables?

Usually! Most shells allow enumerating their environment.

- `csh` uses `set`
- in `bash` you can use `declare -p | grep -v -- ' -x '`
- in `zsh` you can use `typeset -p | grep -v ^export`
- in `fish` you can use `set -S` to get information on all variables and their scope(s)

## Reading

The main point I will stress is _do not use_ `echo $VAR` to check for environment variables! The result of that variable is computed by the shell before
execution, so that will return shell variables that may not be configured for export.

### Your shell's

The `env` and `printenv` commands will display all environment variables exported from your current environment. As this is a straight up command, it does not
matter which shell environment you are running.

### Something else's

Linux, at least, has the option to show environment variables in `ps` output - try `ps eaww` and `ps eww -p1`. There is also `/proc/${pid}/environ` typically.

## Setting

This is shell specific, again - however most shells use `export`. The `csh` shell is a notable exception here, using `setenv`.

## Overriding

If you need to override an environment variable, `env` has a trick up it's sleeve - You can prepend your program invocation with it to override variables like
so:

```
env NAME=VALUE NAME2=VALUE command
```

## Forgetting

This is all shell specific. Most shells let you 'demote' an environment variable into a shell-exclusive one, too.

- `csh`, appropriately, uses `unsetenv`
- in `bash` you can use `declare +x ENVNAME` to turn an environment variable into a shell variable. You may also use `unset ENVNAME` to remove it entirely.
- `zsh` has features similar to bash - `typeset +x ENVNAME` will turn environment variables in to shell variables, `unset ENVNAME` will completely drop it.
- `fish` uses `set -xl ENVNAME` to remove the export scope from a variable. It also uses `set` for erasing variables, via `set -e ENVNAME`.

# Important Environment Variables

## PATH

When your shell is given a line to evaluate, and the first word is not otherwise recognized, the shell will start looking for a _file_ to run that matches the
word. The list of directories checked by the shell is defined in the `PATH` variable. Since there is no convention for passing arrays or lists via environment
variables, this is flattened into a `:` delimited string. Directories are checked linearly, from first to last in the list. Here's a quick demonstration of
that mechanic:

```
$ echo $PATH
/home/rj/Apps/packer-0.12.3/bin:/home/rj/Apps/packer-0.12.2/bin:/usr/bin:/bin:/usr/sbin:/sbin
$ which packer
/home/rj/Apps/packer-0.12.3/bin/packer
$ export PATH=/home/rj/Apps/packer-0.12.2/bin:/usr/bin:/bin:/usr/sbin:/sbin
$ which packer
/home/rj/Apps/packer-0.12.2/bin/packer
$ export PATH=/home/rj/Apps/packer-0.12.2/bin:/usr/bin:/bin:/usr/sbin:/sbin:/home/rj/Apps/packer-0.12.3/bin
$ which packer
/home/rj/Apps/packer-0.12.2/bin/packer
```

Note that the `packer` binary the shell found stayed with the one in the 0.12.2 folder, even after I added 0.12.3 - because I added to the _end_ of `PATH`.

### Other path-type environment variables

There are other environment variables that use this delimited list setup, and they often also have PATH in their names - `GOPATH` and `MANPATH` are examples.

## EDITOR

Remembering that most of these are by convention - `EDITOR` is usually used by anything that needs to get text input before proceeding - notably `git` commit
messages. You don't need to set this to just the name of a program typically - I actually have mine as `gvim -f` to invoke a GUI editor and make sure whatever
started it waits for the editor session to close. However, sometimes you will encounter a [bug](https://github.com/voxpupuli/hiera-eyaml/issues/130)
with this setup. Remember, it's up for every program to decide how to handle environment variables themselves.

## PAGER

Much like `EDITOR`, `PAGER` is used when something is presenting a lot of text and wants to give you the option to interactively 'page' through it. The
desired interaction is slightly different, most programs that are considered "pagers" simply allow for displaying a screenful of text, then having a user
press space to continue on to the next page and such.

## TERM

The `TERM` environment variable is one of the very few environment variables you _do_ take with you. It is used to help identify the correct way to interact
with your console/shell window for interactive programs - the shell itself is an interactive program! If you are having trouble with "full-screen" applications
in your shell sessions, this is a least worth looking at. Unfortunately, the how and why of setting it is also rather arcane, and will not be detailed here.

## HOME

This environment variable refers to your user directory on most systems, it is especially convenient if you need to go to a different directory to run
a process, but want a quick way to refer to files you have created - you can use something like `$HOME/my-cool-file` to get at them from anywhere.

## TMPDIR

If you call `mktemp` or otherwise work with something that needs to buffer to a file, especially if it needs to be a large file, manipulating `TMPDIR` to a
location suited to your needs can be handier than explicitly calling `mktemp` with a template specification.

## TZ

If you are working with a remote system, but you want times on the system to be reflected in your time zone, not wherever the system is, the `TZ` variable is
a convenient hook to override it. For details see [this FreeBSD man page](https://www.freebsd.org/cgi/man.cgi?query=tzset&sektion=3&n=1)
