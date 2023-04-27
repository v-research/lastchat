# Bash Scripting Course - Implementing LASTchat
As an n00bies course on Bash scripting we will implement
a chat to explore the key concepts of Bash.

I read the [Bash Reference Manual](https://www.gnu.org/software/bash/manual/bash.html) to
create this course. You can simply read it and learn it yourself from a much better 
source. Our objective is to learn some Bash by coding.

#### What is Bash?
Our [reference manual](https://www.gnu.org/software/bash/manual/bash.html#What-is-Bash_003f)
says that Bash (Burne-Again SHell) is a *shell*; and a shell is "both a command interpreter and a programming language".
Let's see what it means by creating a first bash script `helloworld.sh`.

#### Shell Scripts
Open an editor and create a file with the following code

> vim helloworld.sh
```
#!/bin/bash

#Now I print hello to the world
echo "Hello World"
```

\# is used for comments but the first line will be 
Bash scripts often begin with #! /bin/bash (assuming that Bash has been installed in /bin), since this ensures that Bash will be used to interpret the script, even if it is executed under another shell.
