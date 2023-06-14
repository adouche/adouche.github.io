## Non-boring routine updates
<sup> January 01, 2023 </sup>

#pkgconfig #CI

Handled routine software updates.

CI is very cool and convenient, because how else would I notice that, with the new libpcap, the wireshark build started to fall?

It started crashing with a very strange error message:

```shell
"ninja: Entering directory `/ix/build/mcpe98eHnOfPdF2O/obj'
ninja: error: 
'/ix/store/iDo8RMwHvMz5h6hQ-lib-pcap/lib/libpcap.a;/ix/store/3KBy5KQKrWQR6EuF-lib-nl/lib/libnl-genl-3.a;/ix/store/
3KBy5KQKrWQR6EuF-lib-nl/lib/libnl-3.a', needed by 'run/tshark', 
missing and no known rule to make it"
```

It really says here that cmake generated an incorrect ninja file - somewhere there, in the list of dependencies of some target, has got such a strange line - 3 static libraries combined through `;`.

In fact, it's pretty clear that cmake just writes there some line that it received from pkg-config, and something unexpected turned out to be there.

Here is a diff in libpcap.pc of two different library versions:

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

Suspicion, of course, caused the Requires.private field [https://people.freedesktop.org/~dbn/pkg-config-guide.html](https://people.freedesktop.org/~dbn/pkg-config-guide.html).

*"Requires.private: A list of private packages required by this package but not exposed to applications. The version specific rules from the Requires field also apply here".*

The most confusing explanation, which IMHO does not explain anything.

I understand this as the beginnings of package management in pkg-config - a set of libraries that should be in the system (because they are needed by dependencies for some .so  in the package).

In the case of static libraries, we get just such a strange hat, from all absolute paths, combined through `;`.

What incorrectly called or understood what, I did not begin to understand, because I already have information about transitive dependencies, no need to duplicate it in .pc files.

So I just took down it to fuck from all .pc - [https://github.com/pg83/ix/commit/bac7f907bc4d8841e4eff9aba93c2a0bd765fc96](https://github.com/pg83/ix/commit/bac7f907bc4d8841e4eff9aba93c2a0bd765fc96).

In general, I try not to use such global text replacements for generated files, but sometimes you can’t do without it, otherwise static build support would turn into huge patches on top of all known build systems.

For example, here is a fix that needs to be applied to all meson.build files - [https://github.com/pg83/ix/blob/main/pkgs/die/c/meson.sh#L107-L109](https://github.com/pg83/ix/blob/main/pkgs/die/c/meson.sh#L107-L109).

Unfortunately, in meson (and in cmake, but not in autoconf!) the author of the build scripts can say "build this as .so, even if the user of the package asked to build statically" without the possibility of override.
