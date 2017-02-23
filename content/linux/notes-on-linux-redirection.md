---
title: "Notes on Linux: Redirection"
date: 2016-08-25
tags: [ "linux" ]
---

Redirection is the ability to manipulate the input and output of commands. This
is most useful when we want to write the result of a command to file. For
instance, let's create a file that lists the contents of my dotfiles directory.

First, let's see what's listed in my dotfiles directory, with `ls -al`. The `ls`
command lists directory contents, and the `-al` flags include hidden `.` files
and displays the output in a long-listing format. This is the output of that:

```bash
total 312
drwxrwxr-x  7 rob rob  4096 Aug 25 08:55 .
drwx------ 42 rob rob 16384 Aug 25 12:34 ..
-rw-rw-r--  1 rob rob   942 Aug 25 12:31 aliases
-rw-rw-r--  1 rob rob  7081 Aug 25 08:57 .bashrc
-rwxrwxr-x  1 rob rob   754 Jul 26 21:08 colours.sh
-rw-rw-r--  1 rob rob 10242 Aug 24 14:22 .dircolors
drwxrwxr-x 21 rob rob  4096 Jun 28 21:03 .fonts
-rw-rw-r--  1 rob rob  1091 Aug 22 13:25 functions
drwxrwxr-x  9 rob rob  4096 Aug 25 12:33 .git
-rw-rw-r--  1 rob rob  1202 Aug 24 15:09 .gitconfig
-rw-rw-r--  1 rob rob   169 Aug 24 14:22 .gitignore
drwxrwxr-x  2 rob rob  4096 Jun 22 20:12 .i3
drwxrwxr-x  2 rob rob  4096 Jun 29 09:01 images
-rw-rw-r--  1 rob rob   262 Jun 20 16:25 .init.vim-rplugin~
-rwxrwxr-x  1 rob rob  4280 Aug 17 20:14 install.sh
-rw-r--r--  1 rob rob 18399 Jul 25 12:04 rc.conf
-rw-rw-r--  1 rob rob     0 Jul 12 16:26 README.md
-rw-rw-r--  1 rob rob  1626 Aug 17 15:08 requirements3.txt
-rw-rw-r--  1 rob rob   774 Aug 17 15:05 requirements.txt
drwxrwxr-x  7 rob rob  4096 Aug 24 14:26 .vim
-rw-rw-r--  1 rob rob 18331 Aug 25 11:01 .vimrc
-rw-rw-r--  1 rob rob  2569 Aug 24 14:22 .Xresources
-rw-rw-r--  1 rob rob    58 Aug 24 14:22 .xsessionrc
-rw-r--r--  1 rob rob   876 Aug 25 11:48 ysb.zsh-theme
-rw-rw-r--  1 rob rob   335 Aug 24 14:22 .zshenv
-rw-rw-r--  1 rob rob  1146 Aug 24 14:22 .zshrc
```

To redirect this output to a file we can use the `>` symbol, pointing towards a
filename. If the file doesn't exist, it will be created.

```bash
ls -al > dotfiles.txt
```

Now if we use the `cat` command to display the contents of the newly created
`dotfiles.txt` we'll see it contains the output from the `ls -al` command.

```bash
% cat dotfiles.txt
total 320
drwxrwxr-x  7 rob rob  4096 aug 25 12:39 .
drwx------ 42 rob rob 16384 aug 25 12:39 ..
-rw-rw-r--  1 rob rob   942 aug 25 12:31 aliases
-rw-rw-r--  1 rob rob  7081 aug 25 08:57 .bashrc
-rwxrwxr-x  1 rob rob   754 jul 26 21:08 colours.sh
-rw-rw-r--  1 rob rob 10242 aug 24 14:22 .dircolors
-rw-rw-r--  1 rob rob     0 aug 25 12:39 dotfiles.txt
drwxrwxr-x 21 rob rob  4096 jun 28 21:03 .fonts
-rw-rw-r--  1 rob rob  1091 aug 22 13:25 functions
drwxrwxr-x  9 rob rob  4096 aug 25 12:34 .git
-rw-rw-r--  1 rob rob  1202 aug 24 15:09 .gitconfig
-rw-rw-r--  1 rob rob   169 aug 24 14:22 .gitignore
drwxrwxr-x  2 rob rob  4096 jun 22 20:12 .i3
drwxrwxr-x  2 rob rob  4096 jun 29 09:01 images
-rw-rw-r--  1 rob rob   262 jun 20 16:25 .init.vim-rplugin~
-rwxrwxr-x  1 rob rob  4280 aug 17 20:14 install.sh
-rw-r--r--  1 rob rob 18399 jul 25 12:04 rc.conf
-rw-rw-r--  1 rob rob     0 jul 12 16:26 readme.md
-rw-rw-r--  1 rob rob  1626 aug 17 15:08 requirements3.txt
-rw-rw-r--  1 rob rob   774 aug 17 15:05 requirements.txt
drwxrwxr-x  7 rob rob  4096 aug 24 14:26 .vim
-rw-rw-r--  1 rob rob 18331 aug 25 11:01 .vimrc
-rw-rw-r--  1 rob rob  2569 aug 24 14:22 .xresources
-rw-rw-r--  1 rob rob    58 aug 24 14:22 .xsessionrc
-rw-r--r--  1 rob rob   876 aug 25 11:48 ysb.zsh-theme
-rw-rw-r--  1 rob rob   335 aug 24 14:22 .zshenv
-rw-rw-r--  1 rob rob  1146 aug 24 14:22 .zshrc
```

Redirection uses standard Unix file descriptors, a method of describing which
input/output resource to access. There are 3 of these file descriptors:

* standard input - commonly referred to as `stdin`, this is the input to a
    command or file resource and is represented by the integer `0`
* standard output - commonly referred to as `stdout`, this is the output from a
    command, such as the listing of files from the `ls` command above. This is
    represented by the integer `1`
* standard error - commonly referred to as `stderr`, this is any error generated
    by a command, and is represented by the integer `2`

Above when we redirected the output from the `ls` command to a file we were
redirecting the `stdout`. This is the default when using `>` and so we didn't
have to specify it, it's the same as doing this:

```bash
ls -al 1> file.txt
```

If we wanted to catch an error we would specify `stderr` instead, represented
with a `2`. Let's create an error and record it in a file. We run a non-existent
command, which gives us a `command not found` error. Running it again we add `2>
error.txt` which redirects the `stderr` to a file. And if we use `cat` to output
the contents of that file, we see the same `command not found` error.

```bash
~/dotfiles master× 13:26
% not_a_command
zsh: command not found: not_a_command

~/dotfiles master× 13:26 C:127
% not_a_command 2> error.txt

~/dotfiles master× 13:26 C:127
% cat error.txt
zsh: command not found: not_a_command
```

If we want we can catch either the `stdout` or the `stderr` from a command
referencing both file descriptors.

```bash
command 1> stdout.txt 2> stderr.txt
```

Or we can write both to a single file using a shorcut...

```bash
command > file.txt 2>&1
```

This basically redirects `stderr` to `stdout` and redirects `stdout` to a file.
I use this often in bash scripts that provision Vagrant machines, redirecting
all the output from commands either to `/dev/null`, which simply makes it
disappear, or to a file. I prefer a *quieter* output on screen, and so redirect
the output from the shell script elsewhere.

Now, the `>` symbol will overwrite any file it is directed at. If we want to
append the output to a file we use `>>` instead. Let's use `echo` to append some
words to a file...

```bash
~/dotfiles master× 13:48
% echo "one" >> file.txt

~/dotfiles master× 13:48
% echo "two" >> file.txt

~/dotfiles master× 13:48
% echo "three" >> file.txt

~/dotfiles master× 13:49
% cat file.txt
one
two
three
```

We can also redirect the `stdin`, using the `<` symbol. For example, we can take
the file we've just created and redirect it to the `sort` command, which, well
sorts stuff.

```bash
~/dotfiles master× 13:49
% sort < file.txt
one
three
two
```

## Further Reading

* [Standard Input and Output Redirection](http://sc.tamu.edu/help/general/unix/redirection.html)
* [Piping and Redirection!](http://ryanstutorials.net/linuxtutorial/piping.php)
* [I/O Redirection](http://www.tldp.org/LDP/abs/html/io-redirection.html)
