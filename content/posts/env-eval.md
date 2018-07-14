---
title: "Environment Variables as a Program Bootstrap"
date: 2018-07-13T15:47:58Z
categories:
 - "software"
 - "nix"
tags:
 - "bash"
 - "ssh"
ogimage: "/reference/bash-256x256.png"
---
# Environment Variables as a Program Bootstrap

Previously, I talked about [environment variables](/posts/environment/) relating to system level environment variables. This article expands
on that with an examples of a program that uses _your_ environment to flexibly communitacte between components, demonstrating ways to
effectively use those.

# An example with ssh-agent

Using `ssh-agent` is a way to improve the usability of encrypted ssh keys - that is, you can have keys stored encrypted on disk, and the agent
program will manage loading and unlocking keys for you. This improves usability of SSH - you no longer need to type a password to unlock your
key for every new ssh invocation. Rather, the agent program holds unencrypted keys in memory and re-locks them after a timeout.

Running `ssh-agent` from your shell produces something like the following output:<a name="ssh-agent-output"></a>

```
SSH_AUTH_SOCK=/tmp/ssh-Y4hQHDP2EEY9/agent.2009; export SSH_AUTH_SOCK;
SSH_AGENT_PID=2010; export SSH_AGENT_PID;
echo Agent pid 2010;
```

Referencing the previous post on environment variables, this output can actually be run as a shell script! It is setting up `SSH_AUTH_SOCK`
and `SSH_AGENT_PID` for other programs that support `ssh-agent`. In typical shells, we _can_ induce program output to be interpreted as
shell code. The keys to this are `$()` and `eval`.

## Command substitution - output into a string

The first step is to get the _text output_ of `ssh-agent` as a string for our shell. That is achieved via the `$()` shell construct, it
runs the program specified inside and returns the standard output to us. Here's a simple example:

```
$ uname -p
x86_64
$ CPU="uname -p"
$ echo $CPU
uname -p
$ CPU=$(uname -p)
$ echo $CPU
x86_64
```

Notice how there was a difference between assigning a variable as a simple name, and running `uname -p`, then assigning the result.

## String evaluation - running the contents of a string

The next step here is `eval` - take a string, and run the contents of that string as if you had typed it in your shell. So, returning to
our [ssh-agent](#ssh-agent-output) example - read the output as a commands you had typed yourself! It sets and exports `SSH_AUTH_SOCK` and
`SSH_AGENT_PID`, then tells you the PID on our screen.

## Gluing it all together

The following example is pretty short, just showing how it all glues together to actually let the program do the work for you:

```
$ printenv SSH_AGENT_PID
$ eval $(ssh-agent)
Agent pid 109
$ printenv SSH_AGENT_PID
printenv SSH_AGENT_PID
```

Remember, the printenv command here does check to see your variable is exported - so now other programs can easily use the SSH agent.
