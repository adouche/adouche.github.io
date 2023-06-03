## Why stal/IX. Part 3
<sup> May 31, 2022 </sup>

#stal/IX #IX #packagemanagement #packagemanager

Prereq: [Part 1](9_Stalix1.md), [Part 2](10_Stalix2.md)

Many times I promised to write why my Linux distribution, and why it is designed this way.

### Part three. About the package manager design

By and large, I had 3 iterations to the package manager design.

**First iteration**

Packages were a description in Python, the graph executor for each node launched Python, which executed this code. Homebrew is about the same, only on Ruby.

Very quickly, I realized that Python is completely unsuitable for this task:

- It is absolutely not composable, you cannot concatenate 2 scripts and get a working script, due to the fact that alignment and indentation are important in Python. Therefore, the task "take this script and replace the build phase with this one" was completely inoperative.

- The second reason is common for both ruby and python, and for everything except posix sh. In these languages, you have to invent some dsl to describe simple actions, that are done in the shell with simple commands:

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

In short, this DSL looked shitty, and I definitely didn't want to give it to the user.

**Second iteration**

Build scripts in Arch style, with meta information in the comments:

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

- The task "take this build script and change a little build() in it or list of dependencies" was still not solved in a reasonable way, so the bootstrap chain contained a lot of redundant code. Well, like, I had to copy the build description for building cmake at the earliest stages, when there is nothing yet, or already for later stages, when the previous cmake and libraries are available.

- Here I was already beginning to strain that I had to repeat the same boilerplate to call meson/cmake/etc. I must say that the same Arch can call cmake directly. And I need to pass 100500 parameters in order to provide a static build, and nix-style paths in the file system (in short, DESTDIR and PREFIX are not the default).

**Third iteration**

At this point, I had jinja - a language for describing and substituting templates, and the packages began to look like they look now.

[https://git.sr.ht/~pg/ix/tree/main/item/pkgs/bin/kitty](https://git.sr.ht/~pg/ix/tree/main/item/pkgs/bin/kitty) - look, t/ contains a template that specializes a common template for some build system, and its two heirs - linux/, darwin/. Check them out before reading the following paragraphs.

It is important to understand here that not a shell script is built from this template, but json, which contains pieces of shell scripts for different phases (build, configure, etc) of the build, and lists of dependencies.

- The tasks "do me the same thing, but with mother-of-pearl buttons" are elegantly solved - expanding and narrowing the list of dependencies, replacing and arbitrary modifications of different blocks.

- It becomes possible to pack all the logic associated with a particular build system into a template, and make it more and more complex.

*Examples:*

* [https://git.sr.ht/~pg/ix/tree/main/item/pkgs/bin/brotli/ix.sh#L7](https://git.sr.ht/~pg/ix/tree/main/item/pkgs/bin/brotli/ix.sh#L7) - and build me brotli, but with an extended list of libraries, because I collect not a lib, but binaries;
* [https://github.com/stal-ix/ix/blob/main/pkgs/bin/curl/ms/ix.sh](https://github.com/stal-ix/ix/blob/main/pkgs/bin/curl/ms/ix.sh) - build curl for me, but with a different http3 library, and fresher sources;
* [https://git.sr.ht/~pg/ix/tree/main/item/pkgs/bin/gdbm/ix.sh](https://git.sr.ht/~pg/ix/tree/main/item/pkgs/bin/gdbm/ix.sh) - build me a binary from libgdbm and add readline to it (pay attention, we are expanding not only a list of libraries, but also configure flags);
* [https://github.com/pg83/ix/blob/main/pkgs/die/c/cmake.sh#L43](https://github.com/pg83/ix/blob/main/pkgs/die/c/cmake.sh#L43) - base template for cmake, see how much squatting I do there, from passing paths to the compiler, before replacing SHARED with STATIC in CMakeLists.txt - [https://github.com/pg83/ix/blob/main/pkgs/die/c/cmake.sh#L81](https://github.com/pg83/ix/blob/main/pkgs/die/c/cmake.sh#L81).
Thanks to this, my build files that the user sees are lean and mean, there is only information on the essence of things (dependencies), and all these squats are hidden from the user.

All things are difficult before they are easy, at some point in time it became clear to me that the template engine very easily solves the problem of the build' arbitrary setup of one or another target.
