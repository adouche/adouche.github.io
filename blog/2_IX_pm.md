## IX package manager: privilege escalation and some other implementation aspects
<sup> January 25, 2022 </sup>

#packagemanager #systemd #system0 #suid

I was reading the source code for PulseAudio and realized that Pottering has been writing the same program his whole life - a graph processing real-time events, and it must be linked at runtime without checking if the graph can actually be executed in real life. This idea is haunting him.

Well, it happens. I am currently working on my 5th build system.

First, a brief introduction for those who have recently joined us.

[IX](https://github.com/stal-ix/ix) is a package manager for Linux/macOS, and a Linux distribution based on it.

I really want as many people as possible to use the package manager (not necessarily right now, but in general), because one person can handle 500 packages (I currently have 300), but more people are needed for more than that.<br>
Therefore, I clearly distinguish between the package manager, which can be used anywhere, even in another Linux distribution, and the distribution built from these packages.<br>
At the same time, I don’t want to lose potential users who somehow like systemd (I generally believe that adults can even in systemd by mutual agreement, although I won't use it for anything).

Therefore, I assume that I will have several "presets" - sets of packages that form a complete idea (type of linking + libc + DE + whatever).<br>
Further, when I say that "I have it set up like this in IX", I mean this preset - [https://github.com/stal-ix/ix/blob/main/pkgs/set/system/0/ix.sh](https://github.com/stal-ix/ix/blob/main/pkgs/set/system/0/ix.sh), or System/0. Everything I do in it may not relate to potential IX on systemd + gnome.<br>
I repeat, I don’t care who uses the package base and how, as long as they help update the packages.

So, the intro is over. Next, about how System/0 will work (or rather, one aspect of its design).

I really want the package manager not to work from root and not to have post-install scripts. And more precisely, I want to be able to install a package by unpacking a folder with its contents, and that's it. This greatly facilitates implementation and simplifies the security model. Similar to how it works in Android.

I've solved almost all the problems associated with this:

* user/group creation
* resolver
* dynamic network configuration

And even kernel settings on a per-package basis.

But one problem remained difficult for me - how to escalate privileges? Yes, sudo. If the package manager doesn’t work as root and there are no post-install scripts, how do you get a suid binary in the system?

Until recently, I solved this problem quite crookedly - a background process that set the setuid bit to some binaries.

The other day I came up with an elegant solution and I hasten to share it.

There is an ssh daemon running on the host (say, via socket activation) as root. And sudo is a simple script that does ssh root@localhost. ssh is set up on a Unix domain socket to avoid broadcasting over the network. Basically this scheme already works.

The more I think about it, the more I like this scheme.

* Privilege escalation in ssh, I think, is even better debugged than in sudo. Because we escalate privileges this way more often, IMHO.
* Login is not by password but by key, one per user, not one per host.
* Password/non-password login remains on the user side (password on the private part of the key), without fragile interference in /etc. 

The scheme works very well - I don't have a single suid binary.
