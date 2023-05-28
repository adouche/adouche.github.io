## Why stal/IX. Part 1
<sup> May 28, 2022 </sup>

#stal/IX #OS

Many times I promised to write about why I created my own Linux distribution and why it is designed this way.

### Part one: Why

There are many things I don't like about modern OSes.
* I don't like the Linux scheduler.
* I don't like the closed parts of macOS, its policies towards applications, and especially towards developer tools (profiling, debugging, etc. - all of this stops working “out of the box”).
* For example, I really don't like that in Unix orphaned processes are inherited by init. I want all my process-spawners not to do double forking so that I can see a completely supervised tree.
* I don't like the complexity that RedHat brings to Linux. For example, systemd - a universal (and very bad!) launcher for a dynamic task graph; the boot management system is completely separate (by the way, it's done correctly in macOS, where there is a separate bootloader, separate socket activation, etc.). PipeWire - a universal handler for multimedia stream graph, and nobody needs it in such quality. All sane (yes-yes, "no true Scotsman") players take an oak ffmpeg without this crappy dynamics and resolving plugin dependencies at runtime.
* I don't like the layering of crap on crap in modern LFS/LSB distributions. Oh, dynamic linking leads to dll hell, well, we'll keep it, but we'll attach ostree and containers for deploying applications. You know, this decision is more of a "face-saving" solution rather than rethinking how it should have been done correctly from the beginning.

This list can go on endlessly, but I think you get the point.

I've been thinking for a while about how to solve the most problems that annoy me with the least effort.

Write my own OS? I get trolled about it approximately once a month. By the way, I think I could create a good OS (someday I need to write about how it would be designed). The only problem is that it may take me about 15 years to be able to run a currently modern graphical application on it, and that's only if I don’t mess up the architecture and tools too much.

I imagine this quite accurately. You can just look at the dynamics of Linux development and extrapolate it based on:
* Using a normal programming language to write 10 times faster than Linux developers do;
* Applying normal architectural practices for kernel development.

Figures on the size of the kernel [https://www.opennet.ru/opennews/art.shtml?num=57260](https://www.opennet.ru/opennews/art.shtml?num=57260).

I roughly understand that Linux became normal around version 2.6, with 5-10 MLOC. If we use a proper language, we need to write somewhere between 0.5-1 MLOC.<br>
In general, everything is predictable and very sad.

As a result, I decided that most of my problems could be solved by a normal Linux distribution that I understand well and can customize to fit my needs.

For example, I can create a daemon that kills all orphaned processes inherited by init gradually and incrementally.<br>
Additionally, I can abandon dynamic linking and DLL hell as a class of problems.

To implement these on the same Fedora - it’s easier to kill myself.

About the overall complexity of the system and how I try to fight with it - well, that's basically what my whole blog is about.

From all of the above, it follows that:
* it is completely pointless to ask me "why not systemd",
* "why not ostree",
* "Why …".

I'm trying to solve the problems known to me in a different way, this is a research hobby-distro, and not a competitor for RHEL in its field.

If I'm satisfied with systemd and dll hell, I'll take Fedora and won't bother.

But I'm curious about "how, in general, it is possible otherwise."
