---
title: "Fully Upgrading Simple Shells in Z Shell"
categories:
  - Zsh
---
We all love the feeling of getting a connection on a Netcat listener from a reverse shell — let's go over the process of fully upgrading them in Zsh.

The first thing we'll do is check which version of Python is available on the target machine (if any):

~~~ bash
which python python3
~~~

Then we can execute the following Python one-liner, appending "3" to the first word if necessary:

~~~ bash
python -c "import pty;pty.spawn('bash')"
~~~

Let's break this command down:

* First, we state that we want to execute Python commands within our terminal with "python -c"

* Then we add quotation marks and insert the Python commands we want to execute

* "import pty" imports a Python module that lets us spawn a pseudo-terminal which tricks commands like **su** (substitute user) into thinking they're being executed inside of a proper terminal

* The semicolon marks the end of the first command

* The second Python command uses a function called "spawn" from the pty module to *spawn* a bash shell

**After executing the command, you'll see the name of the user you're logged in as, the hostname of the machine you're connected to, and your current directory.**

![image](https://user-images.githubusercontent.com/85040841/147182741-75abaf0c-6421-406b-8bae-9f6852c2606e.png)


**Much better!**

## Obtaining an Interactive TTY in Zsh

If we want to use a text editor, take advantage of tab-complete, or move through our command history,

We'd run into some problems if we *only* used the Python one-liner we just went over.

Thankfully, there's a way to obtain an interactive TTY shell using **STTY** options.

If you've followed popular guides on this topic in the past, you likely ran into a few issues as a user of Kali's default shell, **Z Shell**.

For Zsh in particular, there's a specific way the commands have to be executed in order to fully upgrade your Netcat shell.

After executing the Python one-liner, let's put our Netcat session in the background with:

~~~
<CTRL+Z>
~~~

![147170816-adbc8a06-0953-4f21-ab51-52011b776e34](https://user-images.githubusercontent.com/85040841/153110601-7fb8fbd5-af3f-4475-8ac8-283028c529ce.png)

With our session pushed to the background, we can obtain some info on the size of our terminal window (in rows and columns):

~~~ zsh
stty size
~~~

![image](https://user-images.githubusercontent.com/85040841/147183707-ecc19772-f14d-4686-8430-dc2576a80894.png)

In this example, my terminal window has **40 rows and 150 columns**.

Now comes the critical part that is unique to Zsh!

In order to ignore our local terminal's hotkeys and return to the reverse shell, we have to run the following:

~~~ zsh
stty raw -echo;fg
~~~

**As Zsh users, we must execute "stty raw -echo" and "fg" in one line.**

After returning to your Netcat session, press  ***Enter***  to refresh it.

The last task is to assign the shell, terminal type, and STTY size (based on the info we gathered earlier).

~~~ bash
export SHELL=bash
export TERM=xterm-256color
stty rows <num> columns <num>
~~~

![image](https://user-images.githubusercontent.com/85040841/153110416-893fa684-d954-4ec5-ad92-e713a5b46bb2.png)

That's it!

We now have a fully-interactive TTY shell that supports tab-complete, job control, text editors, command history, etc.

## Reference Sheet

~~~ zsh
python -c "import pty;pty.spawn('bash')"
<CTRL+Z>
stty size
stty raw -echo;fg
<Enter>
export SHELL=bash
export TERM=xterm-256color
stty rows <num> columns <num>
~~~
