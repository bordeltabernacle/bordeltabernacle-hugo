---
title: "Notes on Linux: Environment Variables"
date: 2016-08-24
tags: [ "linux" ]
---

Environment variables are a pretty fundamental aspect of working with Linux, and
are often assumed knowledge in blog posts and tutorials. But they're also one of
those things that are so basic they can get overlooked. I have to admit I've
been able to survive using Linux for a while now without really knowing how to
use and manipulate them. So, this is me taking the time to understand them
better.

## Environment? Variable?

The environment is the configuration that is set up every time a new session is
started on your computer, and defines how the system operates.

A variable is a way of representing or storing information. This is usually
achieved using a `KEY` and a corresponding `VALUE`. In the case of a traditional
dictionary the word or phrase is the key, and it's meaning is the value. With
this we can easily access meanings through their corresponding words.

So, environment variables are pieces of information made available to your
computer operating system in order to configure it appropriately.

Linux environment variables are stored in the following format, where it is
possible to store multiple values, separating them with a colon.

```bash
KEY=value1:value2:value3:...
```

## Accessing Environment Variables

To access our environment variables we can use the `env`, `printenv` or `set`
commands. Along with environment variables `set` displays shell variables, shell
functions and local variables. `env` and `printenv` are very similar, displaying
the same information, though with `printenv` you are able to access specific
variables, and with `env` you can provide specific variables to a command in
order to temporarily modify the environment for that command.

```bash
% printenv | less
USER=rob
LANGUAGE=en_GB:en
XDG_SEAT=seat0
XDG_SESSION_TYPE=x11
USER=rob
LANGUAGE=en_GB:en
XDG_SEAT=seat0
XDG_SESSION_TYPE=x11
SSH_AGENT_PID=3928
SHLVL=1
HOME=/home/rob
DESKTOP_SESSION=i3
QT_LINUX_ACCESSIBILITY_ALWAYS_ON=1
GTK_MODULES=gail:atk-bridge
XDG_SEAT_PATH=/org/freedesktop/DisplayManager/Seat0
DBUS_SESSION_BUS_ADDRESS=unix:abstract=/tmp/dbus-w7NbwfF1ee,guid=e8872840b094e30cfe473cae57bd4b7d
QT_QPA_PLATFORMTHEME=appmenu-qt5
MANDATORY_PATH=/usr/share/gconf/i3.mandatory.path
LOGNAME=rob
DEFAULTS_PATH=/usr/share/gconf/i3.default.path
XDG_SESSION_ID=c2
GTK2_MODULES=overlay-scrollbar
PATH=/home/rob/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/usr/local/go/bin:/home/rob/code/go/bin
GDM_LANG=en_GB
XDG_SESSION_PATH=/org/freedesktop/DisplayManager/Session0
XDG_RUNTIME_DIR=/run/user/1000
DISPLAY=:0
LANG=en_GB.UTF-8
XDG_CURRENT_DESKTOP=i3
DESKTOP_STARTUP_ID=i3/rxvt-unicode/3789-6-agnesmartin_TIME13812728
XDG_SESSION_DESKTOP=i3
XAUTHORITY=/home/rob/.Xauthority
XDG_GREETER_DATA_DIR=/var/lib/lightdm-data/rob
SSH_AUTH_SOCK=/tmp/ssh-rR4FnFfhpAGZ/agent.3926
SHELL=/usr/bin/zsh
NODE_PATH=/usr/lib/nodejs:/usr/lib/node_modules:/usr/share/javascript
QT_ACCESSIBILITY=1
GDMSESSION=i3
:
```

Above is the output of `printenv`, piped through `less` for brevity (the `%`
is from my prompt rather than part of the command). To access specific values
we follow `printenv` with the key we want.

```bash
% printenv HOME
/home/rob
```

And we can temporarily alter our environment with `env`, placing it in front
of the command we want to provide the environment variable for, in this case
a simple `printenv`:

```bash
~ 13:35
% printenv HOME
/home/rob

~ 13:35
% env HOME=/another/path printenv HOME
/another/path

~ 13:35
% printenv HOME
/home/rob
```

(Again, the `~ 13:35` is just the time in my prompt rather than anything to do
with the command)

We can also access environment variables with the `echo` command, though when we
do this we have to include a `$` to indicate it is a variable rather than the
literal word...

```bash
~ 13:40
% echo HOME
HOME

~ 13:40
% echo $HOME
/home/rob
```

## Setting & Unsetting Variables

To set an environment variable we can use the same `KEY=value` format to assign
a value to a key...

```bash
~ 13:45
% printenv TERM
rxvt-unicode-256color

~ 13:45
% TERM=xterm

~ 13:45
% printenv TERM
xterm

~ 13:45
% export TERM
```

Setting this only applies to the current shell session, it won't apply to any
other terminal sessions, so we have to `export` the variable. Setting this
overwrites the current value of the variable, which is not always what we want
to do. For instance we may need to add a new directory to the `PATH` variable,
a list of all the directory locations the system will look for commands in. To
append a new location we can do the following...

```bash
~ 13:55
% printenv PATH
/home/rob/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/usr/local/go/bin:/home/rob/code/go/bin

~ 13:55
% PATH=$PATH:/home/rob/code

~ 13:55
% printenv PATH
/home/rob/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/usr/local/go/bin:/home/rob/code/go/bin:/home/rob/code
```

We use the current value of `PATH` as part of the new value, adding on our new
value to the end, separating the two with a colon, and assign it back to the
`PATH` variable. We can now see it at the end of our PATH. Again we would need
to `export` this to make it available to all shell sessions.

As well as system environment variables, we can also create and assign
user-defined variables...

```bash
~ 14:01
% printenv SOME_KEY

~ 14:01 C:1
% SOME_KEY=some_value

~ 14:01 C:1
% export SOME_KEY

~ 14:01
% printenv SOME_KEY
some_value
```

These can often be created for specific program requirements.

To unset an environment variable we can use the `unset` command...

```bash
~ 14:41
% printenv KEY

~ 14:41 C:1
% export KEY=value

~ 14:41
% printenv KEY
value

~ 14:41
% unset KEY

~ 14:41
% printenv KEY

```

## Persisting Environment Variables

These environment variable changes will not survive a system restart and we
don't want to have to type them in every time we start our system. So we have to
include them in a configuration file that is run every time we boot up. On bash
this could be `~/.bashrc`, `~/.bash_profile`, `~/.bash_login` or `~/.profile`,
there's plenty of discussion online about where to keep environment variables. I
use ZSH and have environment variables in both my `~/.zshrc` and `~/.zshenv`.

In my `./zshrc` I set the `LANG` variable and the `TERM` variable:

```bash
export LANG=en_GB.UTF-8
export TERM=rxvt-unicode-256color
```

And in `~/.zshenv` I set my `PATH` variable and the value of my `GOPATH`, a
user-defined variable used by the Go programming language.

```bash
export PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games"
export PATH=$HOME/bin:$PATH

# GOPATH
export GOPATH=$HOME/code/go
```

I've had a few *ohhh, that's why that is like that* moments while learning about
Linux environment variables. It feels good to properly understand what's
going on. The links below have been super helpful...

- [Comptia Linux+ Exam Guide](https://www.amazon.co.uk/CompTIA-Certification-LX0-103-LX0-104-101-400/dp/0071841687/ref=sr_1_1?ie=UTF8&qid=1472046714&sr=8-1&keywords=comptia+linux)
- [computerhope.com](http://www.computerhope.com/unix/uenv.htm)
- [Guide to Unix](https://en.wikibooks.org/wiki/Guide_to_Unix/Environment_Variables)
- [lostsaloon.com](http://www.lostsaloon.com/technology/how-to-view-edit-and-delete-an-environment-variable-in-linux/)
- [superuser.com](https://superuser.com/questions/284342/what-are-path-and-other-environment-variables-and-how-can-i-set-or-use-them)
- [digitalocean.com](https://www.digitalocean.com/community/tutorials/how-to-read-and-set-environmental-and-shell-variables-on-a-linux-vps)
- [cyberciti.biz](http://www.cyberciti.biz/faq/linux-list-all-environment-variables-env-command/)
- [Ubuntu](https://help.ubuntu.com/community/EnvironmentVariables)
- [zsh](http://zsh.sourceforge.net/Intro/intro_3.html)
