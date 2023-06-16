## Developed a portal for Mesa update
<sup> March 07, 2023 </sup>

#xdg-open #portal #mesa #zink

I am being asked why I decided to create my own portal.<br>
Here's the story.

### About xdg-open, first

For a reasonable Linux application, there are two ways to open an external relative to itself file:
1. Call the portal via D-Bus.
2. Call the command line tool xdg-open, passing it the path to the file or URL.

It is assumed that each DE will provide its own portal or its own xdg-open script.<br> 
(I'm simplifying a bit here for ease of explanation.)

Unfortunately, in the wild I only found the implementation from freedesktop, and I disliked it so much that when trying to describe it, only swear words remain
[https://www.freedesktop.org/wiki/Software/xdg-utils/](https://www.freedesktop.org/wiki/Software/xdg-utils/).

(By the way, insane applications also have other paths. For example, KDE-lovers are high from KParts (ab)using, but this is not about that.)

For some time, I had a stub for xdg-open that just opened everything in the browser - 
[https://github.com/pg83/ix/blob/99291c90267d7b690bc39fca7224c0a20b76334c/pkgs/bin/xdg/open/ix.sh#L](https://github.com/pg83/ix/blob/99291c90267d7b690bc39fca7224c0a20b76334c/pkgs/bin/xdg/open/ix.sh#L).

The browser can open almost everything, and that solved 90% of my problems.

On the one hand, it is somewhat embarrassing to give people such a solution, but on the other hand, I didn’t want to violence my brain with freedesktop's xdg-open.

So I decided to use my knowledge of what binaries I have in my distribution and wrote this script - [https://github.com/pg83/ix/blob/main/pkgs/bin/xdg/open/scripts/xdg-open](https://github.com/pg83/ix/blob/main/pkgs/bin/xdg/open/scripts/xdg-open).

It determines the mime type of the passed file and tries to choose the most suitable program from PATH for each known type. And falls back to the browser if nothing is found (there is a "minor" problem here - what if xdg-open called the browser?).

Programs are sorted by their popularity because I assume that less well-known == more specialized for the user, not just installed for no reason.

Since in a statically linked distribution, programs cannot appear from anywhere else except from its package base, I thought that this solution would work for a year or two.

In general, this happened, but I also had to create an xdg portal on top of my xdg-open.

### Why xdg portal?

When updating Mesa to the 23 branch, my Zink OpenGL driver broke for the Epiphany + its WebKitWebProcess bundle (this is a separate process for rendering (and more)).<br>
And it worked great in all other programs. So I decided to build Epiphany + WebKitWebProcess with another OpenGL driver - RadeonSI.

How can you select an OpenGL driver for an application in Mesa?

* Fix crapped xml with configs-settings-hacks for applications that do not work with default settings
[https://gitlab.freedesktop.org/mesa/mesa/-/blob/main/src/util/00-mesa-defaults.conf](https://gitlab.freedesktop.org/mesa/mesa/-/blob/main/src/util/00-mesa-defaults.conf).<br>
Yes, there is strcmp() by application name, so-so solution. I decided to leave this method as a last resort.

* The second way is to intervene with the driver loader in EGL and somehow override its decision.<br>
That's what I wanted to do, but unfortunately, Zink is integrated with Mesa through the ass. It wants not only a driver but also an option to be set when loading the application, which says that Zink must be used. Why? Because instead of passing some settings through the factory with drivers, it was easier for them to check for == "zink" in five places in the code. Especially the "harmful" place - [https://gitlab.freedesktop.org/mesa/mesa/-/blob/main/src/egl/main/eglapi.c#L695](https://gitlab.freedesktop.org/mesa/mesa/-/blob/main/src/egl/main/eglapi.c#L695).<br>
What is written here? It says here that Zink is not a real driver, it uses spares from the OpenGL software renderer in its code, and it needs to be initialized more complexly than just calling the initialization function from the driver.<br>
You can immediately see the hand of the master with the professional motto "and everything else can go to hell after me".<br>
And, therefore, intervening in the driver selection code would not have helped because the selection happens earlier.

* The third way is to set this same variable to different values ​​for different applications.<br>
It seemed like we set this environment variable in the wrapper for Epiphany, and everything works?<br> 
Almost, but not quite.<br> 
Any external programs that we can call to open files that have just been downloaded stop working. Because these are ordinary, good, programs that need a different value for this environment variable, common to the entire rest of the system.

And then, of course, I remembered that epiphany can work in flatpak/snap, which means it can use portals, which means it can call external programs to view files through the portal.<br>
Next, my portal opens the app for viewing in a clean environment that doesn't have the corrupted MESA_LOADER_DRIVER_OVERRIDE environment variable.

(Actually, I cut a little here, and the path to the portal is not exactly the same, but the full version would greatly complicate the story, and would not give more understanding).

Well, further it was a case for small.
