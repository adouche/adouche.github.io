## One more reason for stal/IX
<sup> January 11, 2023 </sup>

#ix #stal/ix #loginshell #interactiveshell

In previous posts I wrote why I started my Linux distribution (Why stal/IX. [Part 1](9_Stalix1.md), [Part 2](10_Stalix2.md), [Part 3](11_Stalix3.md).

In short, a lot of things done in Unix/Linux piss me off, and the easiest way to fix them is to have control over the whole environment!

Recently I was figuring out with a colleague why ssh works weirdly on a machine with stal/IX installed. The "weirdness" was the difference in behavior of ssh daemon in interactive vs. batch mode.

In fact, this weirdness is very easy to explain - it is caused by a confusion between the login shell and the interactive shell!

Unfortunately, these modes often coexist, so there is a temptation to stuff your settings for the interactive shell into the configuration files for the login shell.

But this is fundamentally wrong!

Login shell is, if very roughly, the first shell in the chain for your user. Usually some display manager starts it, like mingetty, agetty, emptty, greetd, sddm, and thousands of them; ssh, putty, telnet, mosh, and so on, are also things that spawn the first shell in the chain!

The interactive shell is where you enter commands from the keyboard.

These are, of course, quite different things.

(By the way, a little aside - if you have cases when the same elements are repeated in PATH - this means that you do not understand the difference between these entities, and sometimes the construction of adding something to PATH works twice, because it's in the wrong file.)

The problem here is further exacerbated by the fact that different shell read different startup files, depending on the mode.

For example, bash reads /etc/profile in both interactive and login mode. And most other shell don't!

In short, trash, waste, and sodomy, layering of habits, misunderstandings, and so on are clearly visible here. Evaluate for example only /etc/environment - [https://superuser.com/questions/664169/what-is-the-difference-between-etc-environment-and-etc-profile](https://superuser.com/questions/664169/what-is-the-difference-between-etc-environment-and-etc-profile) - this is upyachka, born from the fact that sometimes you need to do "login", but without "shell", but variables have to be taken from somewhere!

So, of course, my intention in stal/IX is to do everything "right" **(\*)**, and not to reproduce the hack that everyone is used to!

Why?

Because if this hack suited me, I would install Fedora! And I need better!

**(\*)** Right - of course ssh and others should start the first shell in login mode. Even better - for each shell, the user must figure out which files to put interactive settings (such as completion), and which - login.
