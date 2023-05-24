## Mesa, light of my life
<sup> March 23, 2022 </sup>

#mesa #meson

Mesa, light of my life, sparks at my fingertips. My sin, my love. Me-sa.

To build it, the currently very popular build system Meson was developed, and in fact, it's very sad because Mesa developers know Meson too well and use any of its fucking features.

Actually, Mesa consists of two parts - a plugin loader that implements various state trackers like OpenGL, Vulkan on top of the plugin interface, and the plugins themselves.

As the Mesa authors know Meson too well, building Mesa looks something like this:
* K object files are compiled;
* N archives are built from them, and it is important to note that there are intersections - the same .o can end up in several archives;
* From these K + N artifacts, M final .so are linked. Also with arbitrary intersections of .a/.o files!

This means that each final artifact contains an arbitrary subset of source files.

As long as this is linked into .so files with hidden symbols, it's not a problem except for increasing the size of the final code and compilation time (because some sources are recompiled several times).

If we link statically, we encounter understandable problems - repeated symbols during linking.

When I first built Mesa, I solved this in a very strange way - I dumped all the final artifacts into one bucket and made one mega-library that replaced all the final artifacts.

Overall, this works pretty well except for one very important scenario - when I need to split this giant artifact into two parts - loader and drivers - and make them two physically separate dependencies.

This is necessary to raise the dependency from drivers to end programs, and all intermediate libraries should only depend on the loader. This is an important requirement if we don't want to rebuild the entire tree of libraries needed to build end GUI programs for each set of drivers and, in fact, for a normal binary build cache.

I approached the task of splitting 5 times:
* Split to .o files the final artifact manually. Breaks down from a combinatorial explosion of various ways to compile drivers;
* Make it the way that everything is compiled into several completely independent .a files - everything rested on the defects of the meson build system and the features of the mesa build files - here I tried to manually edit the Mesa build scripts (does not scale for updates), and meson itself (here I was closest to victory, but did not work out).

In short, so far I solved this problem like this:
* Wrote a generic procedure for subtracting one .a from another. This cannot be done by the names of .o files, for the reasons that I have already described above, so everything is done character by character [https://github.com/pg83/ix/blob/main/pkgs/bld/librarian/substr.py](https://github.com/pg83/ix/blob/main/pkgs/bld/librarian/substr.py).
* By sequential subtraction puted the bootloader and driver library in order. 

This is much more aesthetically pleasing than the previous .o sack idea, and works quite well (at least it solves the original problem).
