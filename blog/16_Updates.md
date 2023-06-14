## Non-boring routine updates
<sup> January 01, 2023 </sup>

#pkgconfig #CI

I was doing routine software updates.

The CI pipeline is really cool and convenient because, otherwise, how else would I have noticed that with the new libpcap the Wireshark build started to fail?

It started failing with a very strange error message:

```shell
"ninja: Entering directory `/ix/build/mcpe98eHnOfPdF2O/obj'
ninja: error: 
'/ix/store/iDo8RMwHvMz5h6hQ-lib-pcap/lib/libpcap.a;/ix/store/3KBy5KQKrWQR6EuF-lib-nl/lib/libnl-genl-3.a;/ix/store/
3KBy5KQKrWQR6EuF-lib-nl/lib/libnl-3.a', needed by 'run/tshark', 
missing and no known rule to make it"
```

It says here that CMake generated an incorrect ninja file. Somewhere in the list of dependencies for some target, there is a strange string - 3 static libraries combined with `;`.

Actually, it's pretty clear that CMake just writes some string there that it got from pkg-config, and something unexpected turned out to be there.

Here is the diff in libpcap.pc of two different library versions:

```shell
Name: libpcap
 Description: Platform-independent network 
    traffic capture library
+Version: 1.10.2
+Requires.private: libnl-genl-3.0 
+Libs: -L${libdir} -Wl,-rpath,${libdir} -lpcap
-Version: 1.10.1
-Libs: -L${libdir} -lpcap
```

The Requires.private field is what raised suspicion, of course [https://people.freedesktop.org/~dbn/pkg-config-guide.html](https://people.freedesktop.org/~dbn/pkg-config-guide.html).

*"Requires.private: A list of private packages required by this package but not exposed to applications. The version specific rules from the Requires field also apply here".*

The most convoluted explanation that, IMHO, explains nothing.

I understand this as the beginnings of package management in pkg-config - a set of libraries that must be in the system (because they are needed by dependencies of some .so in the package).

In the case of static libraries, we get just such strange nonsense, from all the absolute paths combined through “;”.

What incorrectly called or understood what, I did not begin to understand, because I already have information about transitive dependencies, no need to duplicate it in .pc files.

So I just took down it to fuck from all .pc - [https://github.com/pg83/ix/commit/bac7f907bc4d8841e4eff9aba93c2a0bd765fc96](https://github.com/pg83/ix/commit/bac7f907bc4d8841e4eff9aba93c2a0bd765fc96).

In general, I try not to use such global text replacements for generated files, but sometimes you can’t do without it, otherwise static build support would turn into huge patches on top of all known build systems.

For example, here is a fix that needs to be applied to all meson.build files - [https://github.com/pg83/ix/blob/main/pkgs/die/c/meson.sh#L107-L109](https://github.com/pg83/ix/blob/main/pkgs/die/c/meson.sh#L107-L109).

Unfortunately, in meson (and in cmake, but not in autoconf!) the author of the build scripts can say "build this as .so, even if the user of the package asked to build statically" without the possibility of override.
