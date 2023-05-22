## About build systems
<sup> March 09, 2022 </sup>

#OSS #buildsystems 

I am saddened by the state of modern OSS build systems. Today, I will discuss one such aspect: every self-respecting modern build system wants to have a package manager within itself.<br>
That is, it provides not only the execution of the build graph of one project but also of all build graphs of all dependencies.

Have you seen cargo? I have written a couple of times about what this claim for comprehensiveness leads to in relation to cargo — the need to wrap all non-cargo dependencies in the cargo build. This looks ugly and leads to problems with diamond-driven dependencies.

The problem is that, despite all the attempts of these build systems authors, they do not become comprehensive, and it is impossible to live within the same ecosystem. Therefore, each such system wraps all external dependencies into itself. This, excuse me, is the square (from the number of build systems) in terms of the complexity of the efforts made.

I already wrote about .wrap files from meson (there is a whole repository for them - [https://mesonbuild.com/Wrapdb-projects.html](https://mesonbuild.com/Wrapdb-projects.html)).

This is something to write about endlessly, here are some very shitty examples:
* nodejs rewrites the build system from v8 to autoconf;
* webkit remakes the build system from ANGLE (Google's implementation of opengl) to CMake;
* chrome vendors a bunch of libraries, I won't describe them individually;
* telegram vendors all its dependencies and builds them in a way that is not compatible with anything.

Fun fact - build systems make life easier for developers, but completely kill the maintainers of this software in the repositories, because de-vendoring all this shit is painful.

I live with this a little easier than any Fedora there. In the case of dynamic linking, vendoring is also path crossing in fs. In the case of static linking, this is at least not visible to the outside, it is enough to de-vendor all sorts of freetype/fontconfig and so on.

Chrome, by the way, is great in this regard, they help de-vendor those parts that are simply necessary (such as font rendering).

Among other reasons, it's painful, because every OSS project, from small to large, invents its own way of vendoring.

Ideally, it would be very cool if the build systems stopped doing such garbage, and agreed with the package systems on the interaction interface. It's actually not very difficult.

Imagine the command:

```shell
mix run lib/z lib/freetype bin/make
     bin/cmake bin/clang/14 bin/ninja -- make -j 16
```

This command will make #realm, in which the specified libraries, specified build tools, and (here it is important!) wrappers for the cc/c++/cpp compiler (well, or rustc, whatever) will be available, which will automatically configure the necessary paths to libraries and header files.

Seems complicated? Well let's simplify it into alias `mixrun=mix run $(cat mix.shell)` - and use it like this:

`mixrun make -j 16` or `mixrun --sanitize=address --opt=-O2 make -j 8`.

Then in the corresponding makefile you don’t need to do autodetect at all, list all sorts of -I / -l-L / etc, but just call simple commands like
`cc -c x.o x.c`.

The same works for cargo, and for any other build system.

The main point is that the project-level build system should not autodetect the presence of dependencies and deliver them. Nix can do that, IX can do that.

The problem is that, at any given time, it is easier for the author of this or that OSS project to crutch up the next vendoring, instead of going to negotiate with all interested parties.

Separately, I note that these kludge package managers are completely crap. I would really like to see how cargo, for example, tries to vendor any lib with settings and data for this lib.
