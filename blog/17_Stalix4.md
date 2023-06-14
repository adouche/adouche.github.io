## One more reason for stal/IX
<sup> January 11, 2023 </sup>

#ix #stal/ix #loginshell #interactiveshell

In previous posts, I wrote about why I started my own Linux distribution. (Why stal/IX. [Part 1](9_Stalix1.md), [Part 2](10_Stalix2.md), [Part 3](11_Stalix3.md)).

In short, many things in Unix/Linux annoy me, and the easiest way to fix them is to have control over the entire environment!

Recently, I was troubleshooting with a colleague about why ssh was behaving strangely on a machine with [stal/IX](https://github.com/stal-ix) installed. The "strangeness" was due to the difference in behavior of the ssh daemon in interactive vs. batch mode.

This strangeness is actually very easily explained - it is caused by confusion between the login shell and the interactive shell!

Unfortunately, these modes often coexist, so there is a temptation to put your settings for the interactive shell into the login shell configuration files.

But this is fundamentally wrong!

The login shell is, roughly speaking, the first shell in the chain for your user. Usually, it is started by some display manager, like mingetty, agetty, emptty, greetd, sddm, and so on; ssh, putty, telnet, mosh, etc. - these are also things that spawn the first shell in the chain!

The interactive shell is where you enter commands from the keyboard.

These are, of course, quite different things.

(By the way, a quick aside: if you have cases where the same elements are repeated in PATH - this means that you do not understand the difference between these entities, and sometimes the construction of adding something to PATH works twice because it is in the wrong file.)

The problem here is further exacerbated by the fact that different shells read different startup files depending on the mode.

For example, bash reads /etc/profile in both interactive and login mode, but most other shells don't!

In short, here we can clearly see trash, frenzy, and sodomy, a layering of habits, misunderstandings, and so on. Evaluate, for example, /etc/environment - [https://superuser.com/questions/664169/what-is-the-difference-between-etc-environment-and-etc-profile](https://superuser.com/questions/664169/what-is-the-difference-between-etc-environment-and-etc-profile) - this is an upyachka born of the fact that sometimes you need to do "login" but without "shell", but variables have to be taken from somewhere!

Therefore, of course, I intend to do everything "right" **(\*)** in [stal/IX](https://github.com/stal-ix), rather than reproduce the hack that everyone is used to!

Why?

Because if I was satisfied with this hack, I would have installed Fedora! And I need better!

**(\*)** Right - of course, ssh and the like should start the first shell in login mode. Even better - for each shell, the user should figure out which files to put their interactive settings (like completion) in and where to put login settings.
