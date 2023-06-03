## Why stal/IX. Part 3
<sup> May 31, 2022 </sup>

#stal/IX #IX #packagemanagement #packagemanager

Prereq: [Part 1](9_Stalix1.md), [Part 2](10_Stalix2.md)

Many times I promised to write about why I created my own Linux distribution and why it is designed this way.

### Part three: About the package manager design

By and large, I had 3 iterations in designing the package manager.

**First iteration**

Packages were Python language descriptions, and a graph executor for each node launched Python, which executed this code. Homebrew is structured similarly, only using Ruby.

Very quickly I realized that Python is not suitable for this task:

- - It is absolutely not composable, you cannot concatenate 2 scripts and get a working script because in Python alignment and indentation are important. Therefore, the task of "take this script and replace the build phase with this" was completely unworkable.

- The second reason is common for both Ruby and Python, as well as everything except POSIX sh. In these languages, you have to invent some DSL to describe simple actions that are done with simple commands in shell:

```
os.setcwd('xxx')
a = os.environ['yyy'] + '/zzz'
os.setcwd(a)
```

vs

```
cd xxx
cd ${yyy}/zzz
```

In short, this DSL looked crappy, and I definitely didn't want to give it to the user.

**Second iteration**

Arch-style build scripts with meta-information in comments:

```
# url https://a.b/c.tgz
# dep lib/c lib/c++
# bld bin/make

build() {
    make -j 8
    ...
}

...
```

This was already better, but:

- The task "take this build script and change the build() or dependency list in it a bit" was still not solvable in a reasonable way, so the bootstrap chain contained a lot of redundant code. For example, I had to copy the build description for building CMake at the earliest stages when nothing was available yet, or for later stages when the previous CMake and libraries were available.

- I was already starting to get annoyed that I had to repeat the same boilerplate for calling Meson/CMake/etc. It's worth noting that Arch, for example, can call CMake directly. However, I need to pass 100500 parameters to ensure static build and Nix-style paths in the file system (in short, DESTDIR and PREFIX are not default).

**Third iteration**

At this point, I discovered Jinja - a language for describing and substituting templates, and the packages began to look like they do now.

[https://git.sr.ht/~pg/ix/tree/main/item/pkgs/bin/kitty](https://git.sr.ht/~pg/ix/tree/main/item/pkgs/bin/kitty) - take a look, there's a template in t/ that specializes the general template for some build system, and two of its inheritors - linux/, darwin/. Look at them before reading the next points.

It is important to understand that a JSON is built from this template, not a shell script. This JSON contains pieces of shell scripts for different build phases (build, configure, etc.) and dependency lists.

- Tasks like "do for me the same thing, but with mother-of-pearl buttons" are elegantly solved: expanding and narrowing the list of dependencies, replacing and making arbitrary modifications to different blocks.

- It becomes possible to pack all the logic related to a particular build system into a template, and make it more and more complex.

*Examples:*

* [https://git.sr.ht/~pg/ix/tree/main/item/pkgs/bin/brotli/ix.sh#L7](https://git.sr.ht/~pg/ix/tree/main/item/pkgs/bin/brotli/ix.sh#L7) - and build me brotli, but with an extended list of libraries, because I collect not a lib, but binaries;
* [https://github.com/stal-ix/ix/blob/main/pkgs/bin/curl/ms/ix.sh](https://github.com/stal-ix/ix/blob/main/pkgs/bin/curl/ms/ix.sh) - build curl for me, but with a different http3 library, and fresher sources;
* [https://git.sr.ht/~pg/ix/tree/main/item/pkgs/bin/gdbm/ix.sh](https://git.sr.ht/~pg/ix/tree/main/item/pkgs/bin/gdbm/ix.sh) - build me a binary from libgdbm and add readline to it (pay attention, we are expanding not only a list of libraries, but also configure flags);
* [https://github.com/pg83/ix/blob/main/pkgs/die/c/cmake.sh#L43](https://github.com/pg83/ix/blob/main/pkgs/die/c/cmake.sh#L43) - base template for cmake, see how much squatting I do there, from passing paths to the compiler, before replacing SHARED with STATIC in CMakeLists.txt - [https://github.com/pg83/ix/blob/main/pkgs/die/c/cmake.sh#L81](https://github.com/pg83/ix/blob/main/pkgs/die/c/cmake.sh#L81).
Thanks to this, my build files that the user sees are lean and mean, there is only information on the essence of things (dependencies), and all these squats are hidden from the user.

All things are difficult before they are easy, at some point in time it became clear to me that the template engine very easily solves the problem of the build' arbitrary setup of one or another target.
