## Сross-compile gracefully using "ix run"
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

### Furthermore!

How prettier cross-compilation became with “ix run”!

Now I can, for example, like this:

```shell
pg-> ./ix run \
    bin/qemu --for_target=aarch64-linux-user \
    bin/convert --target=linux-aarch64 \
    -- qemu-aarch64 '$(command -v convert)'
```

What is written here?

It's written here: "build me a realm that has a host qemu that can run arm binaries, build me an image magick convert under aarch64, and run the convert program in this realm".

What this command displays on the screen:

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

Please note that it is exactly the aarch64 binary “convert" that is run!

In fact, I can manipulate artifacts assembled for different target platforms in one build graph.

What does this give us?

* [https://github.com/pg83/ix/blob/main/pkgs/set/ci/unwrap/ix.sh](https://github.com/pg83/ix/blob/main/pkgs/set/ci/unwrap/ix.sh) - cheap autobuild and CI for other platforms. Really, adding aarch64 to the autobuild took 2 lines in the build files.<br>
[https://github.com/pg83/ix/blob/main/pkgs/set/ci/unwrap/linux/aarch64/ix.sh](https://github.com/pg83/ix/blob/main/pkgs/set/ci/unwrap/linux/aarch64/ix.sh) - a list of what I regularly build under aarch64. There is also gdb, and even graphics programs!

* New opportunities for bootstrap. For example, Go is currently not reproducible in terms of the classical ways (package managers and build systems) because the latest version of Go compiled by the C compiler cannot build code under M1, and is not built under it. I can now approach this problem by writing in the build file for go something like:

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

Raise in this assembly node qemu (which I built myself), linux kernel, go1.4, and run the bootstrap chain there. And it will work, including under host == darwin.

It seems to me now that I have achieved something that no one has done before in open source. All autobuild cross-compiling solutions known to me are based on the fact that someone prepared pre-built tools by hand, so they have a rather small scope.

Overall, "ix run" (or any other metasearch engine) certainly changes the approach to development a lot.

For example, now I'm building my go-portal like this -
[https://github.com/pg83/portal/blob/main/build.sh](https://github.com/pg83/portal/blob/main/build.sh).
```
ix run set/dev/go -- go build
```

"Somewhere out there" a temporary realm is being built, with all the needed environment, in which a freshly compiled go build is run on my project. Overhead - split second.

Do you want cgo, and dependency on some C-library?
```
ix run set/dev/go --cgo lib/gtk/3 -- go build ...
```
I think that in the coming years, some other way to be developed in OSS should go into oblivion, because such metapackage systems are gradually going to the masses.
