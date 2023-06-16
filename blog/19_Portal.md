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

Unfortunately, in the wild, I only found the implementation from freedesktop, and it is so shitty that when I try to describe it, only swear words remain.
[https://www.freedesktop.org/wiki/Software/xdg-utils/](https://www.freedesktop.org/wiki/Software/xdg-utils/)

(By the way, insane applications also have other paths, for example, KDE-lovers are high from KParts (ab)using, I would kill them for this, but this is not about that).

I had a stub for xdg-open for a while that just opened everything in the browser - 
[https://github.com/pg83/ix/blob/99291c90267d7b690bc39fca7224c0a20b76334c/pkgs/bin/xdg/open/ix.sh#L](https://github.com/pg83/ix/blob/99291c90267d7b690bc39fca7224c0a20b76334c/pkgs/bin/xdg/open/ix.sh#L).

The browser can open almost everything, and it solved 90% of my problems.

On the one hand, it’s a bit embarrassing to give away such to people, but I didn’t want to violence my brain through xdg-open from freedesktop either.

Therefore, I decided to take advantage of the knowledge of what kind of binaries I have in general in the distribution, and wrote the following script - [https://github.com/pg83/ix/blob/main/pkgs/bin/xdg/open/scripts/xdg-open](https://github.com/pg83/ix/blob/main/pkgs/bin/xdg/open/scripts/xdg-open).

It determines the mime type of the transferred file, and tries, for each known type, to select the most appropriate program from PATH. Well, it fails into the browser if nothing is found (there is a "minor" problem here - what if xdg-open called the browser?).

Programs are sorted by their popularity. Because I'm assuming that the lesser known program == is more specialized to the user, he installed it not just to, but for a reason.

Since programs cannot appear in a statically linked distribution from elsewhere then its package base, I thought that such a solution would quite live for a year or two.

In general, this happened, but I also had to make the xdg portal on top of my xdg-open.

### Why xdg portal?

When updating mesa to branch 23, my zink opengl driver broke for the epiphany + its WebKitWebProcess bundle (this is a separate process for rendering (and not only)).<br>
And in all other programs, it worked great. So I decided to build epiphany + WebKitWebProcess with another opengl driver, radeonsi.

How can you select an opengl driver for an application in mesa?

* Fix crapped xml with configs - settings - hacks for applications that don't work with default settings.
[https://gitlab.freedesktop.org/mesa/mesa/-/blob/main/src/util/00-mesa-defaults.conf](https://gitlab.freedesktop.org/mesa/mesa/-/blob/main/src/util/00-mesa-defaults.conf)<br>
Yes, it's strcmp() there by application's name, so-so solution. I decided to leave this method as a last resort.

* The second way is to interfere with the driver loader in egl, and somehow override its solution.<br>

  That's what I wanted to do, but, unfortunately, zink is screwed to mesa through the ass.<br>
  It wants not only a driver, but that an option be set when loading the application that says that it is necessary to use zink.<br> 
  Why? Because, instead of pushing some settings through the driver factory, it turned out to be easier for them to check for == "zink" in 5 places in the  code.<br> 

  A particularly "harmful" place - 
[https://gitlab.freedesktop.org/mesa/mesa/-/blob/main/src/egl/main/eglapi.c#L695](https://gitlab.freedesktop.org/mesa/mesa/-/blob/main/src/egl/main/eglapi.c#L695).<br>
  What is written here? It says here that zink is not a real driver, it uses spares from the opengl software renderer in its code, and it needs to be   initialized more difficult than to call the initialization function from the driver.<br>
  You can immediately see the hand of the master with the professional motto "and everything else can go to hell after me".<br>
  Well, therefore, interfering with the driver selection code simply would not help, because the selection happens earlier.

* The third way - to set this one variable into different values for different applications.<br>
It would seem that in the wrapper for epiphany we set this environment variable, and everything works?<br> 
Almost, but not quite.<br> 
Any external programs that we can call to open files that have just been downloaded stop working. Because these are ordinary, good, programs that need a different value for this environment variable, common to the entire rest of the system.

And then, of course, I remembered that epiphany can work in flatpak/snap, which means it can use portals, which means it can call external programs to view files through the portal.<br>
Next, my portal opens the app for viewing in a clean environment that doesn't have the corrupted MESA_LOADER_DRIVER_OVERRIDE environment variable.

(Actually, I cut a little here, and the path to the portal is not exactly the same, but the full version would greatly complicate the story, and would not give more understanding).

Well, further it was a case for small.
