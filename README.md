# Bash Scripting Course - Implementing LASTchat
As an n00bies course on Bash scripting we will implement
a chat to explore the key concepts of Bash.

I read the [Bash Reference Manual](https://www.gnu.org/software/bash/manual/bash.html) to
create this course. You can simply read it and learn it yourself from a much better 
source. Our objective is to learn some Bash by coding.

## Introduction: What is Bash?
Our [reference manual](https://www.gnu.org/software/bash/manual/bash.html#What-is-Bash_003f)
says that Bash (Burne-Again SHell) is a *shell*; and a shell is "both a command interpreter and a programming language".
Let's see what it means by creating a first bash script `helloworld.sh`.

### Shell Scripts
A shell script is a text file containing shell commands.
So, open an editor and create a shell script

```
vim helloworld.sh
```
```
#!/bin/bash

#This is a comment: Now I print hello to the world!
echo "Hello World"
```

Give to the user the permission to execute the script (instead of just read and write permissions)
with the `chmod` command. [PAUSE] I won't describe all the commands in details but you can also run `man command` (e.g., `man chmod`)
to have a description of the command with all the possible argument (and, sometimes, examples too).
![image](https://user-images.githubusercontent.com/14936492/234990105-3f00b829-86af-47d6-8b7b-673070bbaa43.png)

To execute a Bash script you just type `./<name_of_the_script>` (substituing `<name_of_the_script>` with, e.g., `./helloworld.sh`).
The file extension is usually `.sh` but you can use any extension (or even none) and the script will be execute nontheless.

\# is used for comments but the first line of this script because: (assuming
that Bash has been installed in /bin), since this ensures that Bash will be
used to interpret the script, even if it is executed under another shell.

The `echo` simply prints what we want. Where? On the STDOUT (the standard output)
which is usually the screen. Later will describe this in more in details.

### Echo in Red
What if we want to add some color to our hello world?
You can tell echo to color its ouput by adding acolor code to the string you echo.
Here are some colors:
```
\e[0m #color default
\e[0;31m #color red
\e[0;32m #color green
```
Which you can use by using the command `echo -e` where the `-e` arg: "enable interpretation of backslash escapes" (from `man echo`).
```
echo -e "\e[0;32m Hellow \e[0m Colored \e[0;31m World!"
```
![image](https://user-images.githubusercontent.com/14936492/234993194-7300a752-5efa-411a-8a04-5768c8e664b0.png)

## LASTchat v0.1
Enough with the introduction, let's write a chat!

### Netcat (nc)
First, I want my chat to exchange messages over a network so that I can chat with people in my network.
The command `nc -l 4444` creates a TCP listeners (server) on the TCP port `4444`. This means that you 
So, if you run it, you can open another shell and write data to that TCP listener with another netcat instance (client) with `nc 127.0.0.1 4444`.

![image](https://user-images.githubusercontent.com/14936492/234994689-22a1750a-4127-4b6b-9a9a-2b2da466dcb7.png)

Great! We have a chat. DONE! :P
You don't believe me? You can use Wireshark to inspect what the `nc` does:
![image](https://user-images.githubusercontent.com/14936492/234995560-2c621d6b-34ad-40da-bc32-25cbfca86028.png)

But:
- [BUG#1] we have a problem: **if you try to use multiple netcat clients** to communicate with listene on the TCP port 4444.
Try to open another nc client and the messages won't be displayed on the server. You can inspect what is happening with Wireshark.
2. [BUG#2] We have another problem: when the client is closed (e.g., by pressing `Ctrl+d`) the server shuts down.

### While loops
You can read about [loops on the reference manual](https://www.gnu.org/software/bash/manual/bash.html#Looping-Constructs)
but a while loop like this one:
```
#!/bin/bash

while [[ true ]]
do
  nc -l 4444  
done
```
loops on the code in between the `do`-`done` keywords since the guard which is checked to stop the while 
loop is what is inside `[[ true ]]` and true is never false :) and the loop never ends.
But you want make your computer explode since, once the `nc -l 4444` is executed then the
script stops and waits for `nc` to finish. 

Why are we doing this? Because we can connect with the client `nc 127.0.0.1` and, once we are done sending
messages to the server we can quit (`Ctrl+d`) the client without killing the server! Problem 2. solved!

## Redirections (LASTchat v0.2)
What if we now want multiple clients to connect to the chat server?
Can we somhow tell netcat to send a message and then just close by itself so that I can always run a client and write to the serve?
The argument (see `man nc`) `-N` does that as it "shutdown the network socket after EOF on the input."
But how to we specify which message to send through it?

In Bash you can "redirect" the output of a command as the input of another one.
So, if we want to redirect the output of the `echo "message"` to 
`nc -N 127.0.0.1 4444` we can simply run the following code.
```
echo "message" | nc -N 127.0.0.1 4444
```
The echo prints the message to the STDOUT but, instead of being sent to the screen,
it is redirect-ed to the STDIN (the standard input) of the command `nc`.

You can read more on redirection in the [referece manual](https://www.gnu.org/software/bash/manual/bash.html#Redirecting-Output)
But you can see the 'pipe' command `|` as taking the output of its left hand side and 
sending it as input to what it's on his right hand side.

Now copy the above while loop into a script called `chat_server.sh` and run it.
Now you can send messages to your server whenever you want with the command above, and the server will always be listening.

- [BUG#3] Chats are not persistent and when the server is closed the chat is gone
- [BUG#4] Clients don't see the messages of the chat! Only the server does
- [BUG#5] There can be issues if two commands are sent at the very same time.

## Redirections to Files (LASTchat v0.3)
To make the chat persistent we can save it into a log file.
As you can redirect an `echo "message"` output into another command (well, process but we are going to discuss processes later on) as input, you can also redirirect it into a file.
```
echo "message" > chat.log
```
And you can test this by using `cat chat.log` to output the content of a file to the STDOUT (see `man cat`).
![image](https://user-images.githubusercontent.com/14936492/235002992-2bca2c7c-0268-4eb2-b3db-62c1c769d2ec.png)

We can now improve our chat server `chat_server.sh` as
```
#!/bin/bash

while [[ true ]]
do
  nc -l 4444 >> chat.log
done
```
I use `>>`, instead of the single `>` because:
- `> file` writes a new `file` (or overwrites the whole conent of a file with the same name)
- `>> file` appends at the end of `file`

and we solved [BUG#3]

## Command Substitution (LASTchat v0.2_logging)
Chat persistency often requires some timestamps so that one knows when the logged string has been logged or, in our case,
when the message has been sent. We can add a timestamp to each message by requiring the client to do so when they send a message.
We can obtain a timestamp with the command `date +FORMAT` where FORMAT can be (see `man date` for more FORMATs) %D-%r for DATE-TIME.

![image](https://user-images.githubusercontent.com/14936492/235004751-33bf40c8-c9a4-449b-881d-e548b62d6e9c.png)

And create a `chat_client.sh`
```
#!/bin/bash

echo "$(date +%D-%r) message" | nc -N 127.0.0.1 4444
```

The syntax `$(command)` is called [command substitution](https://www.gnu.org/software/bash/manual/bash.html#Command-Substitution)
and the `command` within the parenthesis is executed and its output substitues `S(command)`.

## Variables (LASTchat v0.3)
[BUG#4] is critical, clients need to see the chat!
We can create a script for the client and re-use the idea of the infinite while loop
so that a client becomes persistent too and I can easily send mulitple messages.
Well, we can't really as the following code would constantly send the same message
over and over.
```
#!/bin/bash
while [[ true ]]
do
  echo "$(date +%D-%r) message" | nc -N 127.0.0.1 4444
done
```

But we can ask the person running the client whether they want to send a message or just wait by using `read var`
that takes a user input and stores into a variable `var`.

![image](https://user-images.githubusercontent.com/14936492/235006748-8475cea0-e711-4dc3-a98b-763bc14b515b.png)

Variables are created by using their name as in `var="ciao"` but their values can be accessed to by using the
keyword `$` a in `$var`. The `echo` will
- print the value of a variable `$var` if using the [double quotes](https://www.gnu.org/software/bash/manual/bash.html#Double-Quotes) `echo "$var"`
- with [single quotes](https://www.gnu.org/software/bash/manual/bash.html#Single-Quotes) it won't expand the `$var` syntax into its value but simply output the string `$var` as it is (try `echo '$var'`)

```
#!/bin/bash
while [[ true ]]
do
  read message #read input from user
  echo "$(date +%D-%r) $message" | nc -N 127.0.0.1 4444 #message is now a variable
done
```
![image](https://user-images.githubusercontent.com/14936492/235008652-b7ecc056-0a3f-47c0-81bc-9cb344de052e.png)
However, clients still don't talk to other unless they look at the server.

## IF Statement
TBD

## Special Variables
TBD

## Arithmetics
TBD

## Make a Rudimentary GUI with Screen
#TBD
