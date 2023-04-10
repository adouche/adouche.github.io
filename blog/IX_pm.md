## IX package manager: privilege escalation and some other implementation aspects
<sup> January 25, 2022 </sup>

#packagemanager

I been reading PulseAudio sources. I realized that Pottering has been writing the same program all his life - a graph for processing some thread of events in real time, and it is imperative that the graph be connected at runtime, and without checking that this graph can be executed in real life. Well, this idea haunts him.

All right, it happens. Some are writing the 5th build system already.

First, a small introduction, for those who joined recently.

[IX](https://github.com/stal-ix/ix) is a package manager for linux/macos and a Linux distribution based on it.

I really want as many people as possible to use the package manager (not right now, but in general), because 1 person will pull 500 packages (I have 300 now),  for more packages more people are needed.<br>
Therefore, I make a very clear distinction between a package manager that can be used anywhere and any way, even on another Linux, and a distribution built from those packages.<br>
Wherein, for example, I don’t want to lose potential users who, for some reason, like systemd (I generally think that adults, by mutual agreement, can also in #systemd, yes, although myself won’t for anything).

Therefore, I assume that I will have several "presets" - sets of packages that form a complete idea (type of linking + libc + DE + whatever).<br>
Further, when I say that "I have something like this in IX", I mean this preset here - [https://github.com/stal-ix/ix/blob/main/pkgs/set/system/0/ix.sh](https://github.com/stal-ix/ix/blob/main/pkgs/set/system/0/ix.sh), or System/0 #system0. Everything I do in it may not be relate to potential IX on #systemd + gnome.<br>
I'll repeat, I don’t care who and how uses the package database, as long as they help update the packages.

So, the intro is over. Further, about how System / 0 will be designed (or rather, about one of the implementation aspects).

I really wish that the package manager didn't run as root, and that it didn't have postinstall scripts. More precisely, I want the package to be able to be installed by unpacking the folder with its contents, and that's it. This greatly facilitates implementation, and simplifies the security model. Let's say like in Android.

I solved almost all problems related to this:

* adding users/groups
* resolver
* dynamic network setting

Yes, even per-package kernel settings.

But there was one problem that remained difficult for me. How to escalate privileges? Yes, sudo. If the package manager doesn’t work as root, there are no postinstall scripts, then how to get the #suid binary in the system?

At first, I solved this problem rather crookedly - a background process that set the setuid bit to some binaries.

Then I came up with an elegant solution, and I hasten to share it.

The host is running (say, via socket activation) an ssh daemon, as root. And sudo is a simple script that does ssh root@localhost. ssh is up on a unix domain socket so as not to shine on the network. Basically, this scheme is already working.

The more I think, the more I like this scheme:

* Privilege escalation in ssh is, I think, debugged even better than in sudo. Because so we escalate privileges IMHO more often;
* Login not by password, but by key, 1 per user, not one per host;
* Password/passwordless login remains on the user's side (password on the private part of the key), without fragile interference in /etc.

The scheme works very well - I don't have any #suid binaries.
