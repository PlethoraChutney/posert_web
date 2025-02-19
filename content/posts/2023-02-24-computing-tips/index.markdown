---
title: Computing Tips
author: Rich Posert
date: '2023-02-24'
slug: []
categories: ['Tips']
tags: ['Computation']
weight: 1
---

I'm writing this to collect some computing tips I've accumulated while being
a graduate student at OHSU. I'll divide them into "Generally useful" and "OHSU
specific", since some of them are particular to how OHSU has set things up.

# Generally useful

## Setting things up

Take a lot of notes when you set things up. Eventually, no matter how permanent
your position is (and if you're a grad student, it's very much not permanent),
you'll need to leave behind detailed instructions on how to maintain things.
You'll save yourself a lot of work if you've written these out ahead of time,
and if you think about what you're doing before you actually do it you might
set up a more maintainable infrastructure than if you do what I did, and say,
"I'll set this up as quickly as I can right now, and fix it later if it breaks".

## SSH

### cryoEM

I recommend, if you're doing cryoEM, that you make extensive use of functions
and aliases! Changing from thinking about "how do I want this mask to look"
to "how do I upload this file and where does it go" is [more expensive than
you might think](https://en.wikipedia.org/wiki/Task_switching_(psychology))!

I have a bunch of aliases and functions that I use all the time.
These time saving functions go into your `.bashrc` (or, for me, `.zshrc`).
Particularly things that require
me to remember paths on remote machines get aliases. For instance, I have an
alias I use to upload a mask:

```
putmask () {
	scp $1 posert@troll.ohsu.edu:~/masks/
}
```

and another one I use to download cisTEM volumes and rename them:

```
getcis () {
        scp posert@exahead1.ohsu.edu:/home/groups/BaconguisLab/posert/$1/cistem-project/Assets/Volumes/$2 $3
}
```

I also have a more complicated (and useful) function to download, organize, and
rename RELION jobs just by their job number, which you can check out
[here](https://github.com/PlethoraChutney/dotfiles/blob/master/getrel).



### Tunneling

If you're not familiar with the concept of a tunnel, it's a way of setting up
a port on your machine that goes over your SSH connection and plugs into a port
on the remote machine. Thename tunnel is quite descriptive: it uses your SSH connection
as a "tunnel" to get from you to your target.

For instance, I run `ssh -L 8080:localhost:80 posert@troll.ohsu.edu` to create a
tunnel from my computer's port 8080 to our workstation's port 80. Port 80 is used
for HTML traffic, so now if I want to access something like `http://troll.ohsu.edu/example`
I can go to `http://localhost:8080/example` without needing to be on OHSU's VPN.

Another example: I have set up an alias in my `.zshrc` file which creates a tunnel
from my computer's port 11312 to our workstation's port 39000. This tunnel lets me
access cryoSPARC without needing to be on the network, just by typing cryoSPARC.

```
alias cryosparc="ssh -f -N -T -L 11312:localhost:39000 posert@troll.ohsu.edu"
```

The `-f`, `-N`, and `-T` options send ssh to the background, stop ssh from
running a command on the remote machine, and prevent terminal allocation respectively.
Basically, this preserves resources and prevents you from having to open a new
terminal window by making this ssh connection "tunnel only".

### Use .ssh/config

I didn't know about `.ssh/config` until very recently! It's a really nice tool
to make ssh-ing around easier. One thing it's great for is if, like OHSU, your
employer only makes a single server publicly accessible. You have to SSH into
that server, then continue from there to your server of choice. This would make
things like copying files, opening tunnels, etc., quite annoying. That's
where the config file comes in!

```
Host acc.ohsu.edu
    ProxyJump none
    ForwardX11 yes
    ForwardX11Trusted yes
    ServerAliveInterval 60
    ForwardX11Timeout 240h

Host *.ohsu.edu
    ProxyJump acc.ohsu.edu
    ForwardX11 yes
    ForwardX11Trusted yes
    ServerAliveInterval 60
    ForwardX11Timeout 240h

```

This file sets up a "proxy jump". Whenever I try to ssh into any address
ending in `.ohsu.edu`, my computer knows that it first needs to connect to `acc.ohsu.edu`
and from there go to my final destination.

There's another nice trick I've included in my config file:

```
Host *
    ControlPath /tmp/ssh-%r@%h:%p
    ControlMaster auto
    ControlPersist yes
```

This keeps any ssh connection I open alive, so that when I SSH into OHSU, or
our workstation, or wherever else, I only need to input my password to open that first
connection. Everything else uses the same SSH connection. Copying multiple files,
opening multiple tunnels, it's all just one password!
