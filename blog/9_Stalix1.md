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

This list is endless, but I think you get the point.

I've been thinking for a while about how, with the least amount of effort, to solve the most problems that annoy me.

Write my own OS? I'm trolled with this about once a month, and, by the way, I would make a good OS (somehow I need to write how it would be designed). The only problem is that I will launch a modern (currently) graphical application there in 15 years, and then, if I don’t get too fucked up with architecture and tools.

I imagine this quite accurately, you can just look at the dynamics of Linux development, and extrapolate it based on:
* take a normal language to write 10 times faster than Linux developers do;
* take the normal architectural principles of building the kernel.

Figures about the kernel size [https://www.opennet.ru/opennews/art.shtml?num=57260](https://www.opennet.ru/opennews/art.shtml?num=57260).<br>
I roughly understand that Linux became ok somewhere around 2.6, 5 - 10 MLOC. If we take a normal language, we need to code somewhere 0.5 - 1 MLOC.<br>
In general, everything is predictable, and very sad.

In the end, I decided that most of my problems can be solved by a normal Linux distribution, which I understand well, and can "bend" in the most appropriate way.<br>
For example, having created a demon, which smites all those nailed to the init orphan process, gradually and incrementally.<br>
For example, refusing dynamic linking and dll hell as a CLASS of problems.<br>
How to make it in the same Fedora - you can kill yourself.

About the overall complexity of the system, and how I'm trying to struggle with it - well, about this, in fact, there is my entire blog.

From all of the above it follows:
* it's completely pointless to ask me "why not systemd"

* "why not ostree"

* "Why ...

I'm trying to solve the problems known to me in a different way, this is a research hobby-distro, and not a competitor for RHEL in its field.

If I'm satisfied with systemd and dll hell, I'll take Fedora and won't bother.

But I'm curious about "how, in general, it is possible otherwise."
