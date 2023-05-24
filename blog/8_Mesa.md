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

I approached the task of splitting about 5 times:
* Manually split the final artifact by .o files. Breaks down due to the combinatorial explosion of different ways to build drivers;
* Make everything build into several completely independent .a files - everything was up against the defects of the meson build system and the peculiarities of the mesa build files - here I tried to manually edit the mesa build scripts (not scalable on updates), and meson itself (here I was closest to victory, but it didn't work out).

In short, for now I’ve solved this problem in the following way:
* Wrote a generic procedure for subtracting one .a from another. It can't be done by .o file names for the reasons I described above, so everything is done symbol by symbol [https://github.com/pg83/ix/blob/main/pkgs/bld/librarian/substr.py](https://github.com/pg83/ix/blob/main/pkgs/bld/librarian/substr.py);
* By subtracting sequentially, I fixed the loader and driver library. 

This is much more aesthetic than the previous idea with a bag of .o files and works well enough (at least, it solves the original problem).
