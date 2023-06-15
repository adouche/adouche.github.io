## Cross-compile gracefully using "ix run"
<sup> February 27, 2023 </sup>

#ix #cross #cross_compilation #bootstrap #realm

I wrote [here](7_Build_systems.md) about how cool it would be to create a realm with one command, in which the specified libraries, specified build tools and wrappers for the compiler will be available, which will automatically configure the necessary paths to libraries and header files.

### Invented - done!

```shell
  ...
  HOSTCC  scripts/basic/fixdep
  HOSTCC  scripts/kconfig/mconf.o
  HOSTCC  scripts/kconfig/lxdialog/checklist.o
  HOSTCC  scripts/kconfig/lxdialog/inputbox.o
  HOSTCC  scripts/kconfig/lxdialog/menubox.o
  HOSTCC  scripts/kconfig/lxdialog/textbox.o
  HOSTCC  scripts/kconfig/lxdialog/util.o
  HOSTCC  scripts/kconfig/lxdialog/yesno.o
  HOSTCC  scripts/kconfig/confdata.o
  HOSTCC  scripts/kconfig/expr.o
  HOSTCC  scripts/kconfig/lexer.lex.o
  HOSTCC  scripts/kconfig/menu.o
  HOSTCC  scripts/kconfig/parser.tab.o
  HOSTCC  scripts/kconfig/preprocess.o
  HOSTCC  scripts/kconfig/symbol.o
  HOSTCC  scripts/kconfig/util.o
  HOSTLD  scripts/kconfig/mconf
configuration written to .config

*** End of the configuration.
*** Execute 'make' to start the build or try 'make help'.

> ix run set/menuconfig -- make HOSTCC=cc menuconfig
```

Previously, to run menuconfig for kernel settings, I would build the kernel through [IX](https://github.com/stal-ix/ix), press ctrl-c, go to the remaining build folder, and there, in the configured environment, run make menuconfig.<br>
It was okay for me, but of course, it's embarrassing to give that away to others.

I implemented the promised feature of the ability to run a command in an arbitrary realm. It became convenient.

### The further - the more!

Cross-compiling with "ix run" has improved.

Now I can, for example, run:

```shell
pg-> ./ix run \
    bin/qemu --for_target=aarch64-linux-user \
    bin/convert --target=linux-aarch64 \
    -- qemu-aarch64 '$(command -v convert)'
```

What is written here?

It's written here: "Build me a realm with QEMU host capable of running Arm binaries, build me ImageMagick convert for AArch64, and run the convert program in this realm".

What does this command output on the screen:

```shell
READY /ix/store/uFlUrE6DQMb3SC2l-rlm-ephemeral/touch
Version: ImageMagick 7.1.0-58 Q16-HDRI aarch64  
    https://imagemagick.org
Copyright: (C) 1999 ImageMagick Studio LLC
License: https://imagemagick.org/script/license.php
Features: Cipher DPC HDRI 
Delegates (built-in): bzlib fontconfig 
    freetype heic jng jp2 jpeg jxl lcms 
    openexr pangocairo png raw tiff 
    webp zlib
Compiler: gcc (4.2)
Usage: convert [options ...] 
    file [ [options ...] 
    file ...] [options ...] file
```

Note that it is exactly the AArch64 binary convert that is run!

Essentially, I can manipulate artifacts built for different target platforms in one build graph.

What does this give us?

* [https://github.com/pg83/ix/blob/main/pkgs/set/ci/unwrap/ix.sh](https://github.com/pg83/ix/blob/main/pkgs/set/ci/unwrap/ix.sh) - cheap auto-build and CI for other platforms. Adding AArch64 to auto-build only took two lines in the build files.<br>
[https://github.com/pg83/ix/blob/main/pkgs/set/ci/unwrap/linux/aarch64/ix.sh](https://github.com/pg83/ix/blob/main/pkgs/set/ci/unwrap/linux/aarch64/ix.sh) - the list of what I regularly build for AArch64 including GDB and even graphical programs!

* New possibilities for bootstrapping. For example, Go is currently not reproducible in terms of classic methods (package managers and build systems), because the latest version of Go compiled with a C compiler cannot build code for M1 and cannot be compiled on it. Now I can approach this problem by writing something like this in the Go build file:

<!-- {% raw %} -->

```shell
{% block bld_tool %}
bin/qemu(for_target=linux-x86_64)
bin/kernel(target=linux-x86_64)
bin/busybox(target=linux-x86_64)
bin/go/1.4/(target=linux-x86_64)
{% endblock %}
```
<!-- {% endraw %} -->

Launch QEMU (which I built myself), Linux kernel, Go 1.4 in this build node and run the bootstrap chain there. And it will work, including under host == darwin.

It seems to me now that I have achieved something that no one has done before in open source. All auto-build cross-compiling solutions known to me are based on someone manually preparing pre-built tools, so they have a rather small scope.

In general, "ix run" (or any other meta-search system) certainly changes the approach to development a lot.

For example, now I build my Go portal like this -
[https://github.com/pg83/portal/blob/main/build.sh](https://github.com/pg83/portal/blob/main/build.sh).
```
ix run set/dev/go -- go build
```

"Somewhere out there" a temporary realm is built with all the necessary environment in which a freshly compiled go build is run on my project. Overhead - fractions of a second.

Want cgo and a dependency on some C library?
```
ix run set/dev/go --cgo lib/gtk/3 -- go build ...
```
I think that in the coming years, some other way of developing in OSS should go into oblivion because such meta-package systems are gradually becoming more popular among the masses.
