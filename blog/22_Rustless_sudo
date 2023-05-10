## Rustless sudo: exploring secure alternatives for suid binaries
<sup> May 11, 2023 </sup>

#stalix #security #suid #sudo

[https://www.memorysafety.org/blog/sudo-and-su/](https://www.memorysafety.org/blog/sudo-and-su/)

Some no-names decided to hype up and rewrite sudo/su in Rust. They are not the first, nor the last.

While I was reading the text, a brilliant idea came to mind about the security of suid binaries. I've been having a lot of good ideas about security lately!

Let's compile important suid binaries like sudo with sanitizers and release them in production?<br>
Seriously, let's compile them with asan, msan, ubsan, and roll them out.

Instead of unnecessarily escalating privileges, they will just crash with a normal panic(). These utilities don't do anything useful anyway, so a slight slowdown won't harm.

I would have already implemented this in [stal/IX](https://github.com/stal-ix) but there is one "but": sanitizers do not work when using static linking and musl - 
[https://wiki.musl-libc.org/open-issues.html](https://wiki.musl-libc.org/open-issues.html).

The following solutions come to mind for securely running suid binaries:

* Run under valgrind. Unfortunately, valgrind itself must be suid bit for this to work, making this solution impractical.
* Compile binaries for an architecture with tagged memory [https://en.wikipedia.org/wiki/Tagged_pointer](https://en.wikipedia.org/wiki/Tagged_pointer), 
and interpret the result, for example, using QEMU. This solution has exactly the same problems as valgrind.
* Compilation in WASM. The most promising solution, given the availability of a large number of simple, interpretable, embeddable runtimes - 
[https://github.com/appcypher/awesome-wasm-runtimes](https://github.com/appcypher/awesome-wasm-runtimes):<br>
[https://github.com/wasmx/fizzy](https://github.com/wasmx/fizzy);<br>
[https://github.com/wasm3/wasm3](https://github.com/wasm3/wasm3).
