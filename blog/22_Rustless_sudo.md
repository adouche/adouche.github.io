## Rustless sudo: running suid binaries
<sup> May 11, 2023 </sup>

#stalix #security #suid #sudo #mem-safe #memory_safe

[https://www.memorysafety.org/blog/sudo-and-su/](https://www.memorysafety.org/blog/sudo-and-su/)

Some no-names decided to hype up and rewrite sudo/su in Rust. They are not the first, nor the last.

While I was reading the text, a brilliant idea came to mind about running suid binaries in a memory safe manner. I've been having a lot of good ideas about security lately!

Let's compile important suid binaries like sudo with sanitizers and release them in production?<br>
Seriously, let's compile them with asan, msan, ubsan, and roll them out.

Instead of unnecessarily escalating privileges, they will just crash with a normal panic(). These utilities don't do anything useful anyway, so a slight slowdown won't harm.

I would have already implemented this in [stal/IX](https://github.com/stal-ix) but there are two "buts": 
* Sanitizers do not work when using static linking and musl - 
[https://wiki.musl-libc.org/open-issues.html](https://wiki.musl-libc.org/open-issues.html);
* There is not a single suid binary in [stal/IX](https://github.com/stal-ix).

The following solutions come to mind for memory-safe running of suid binaries: 

* Run under valgrind. Unfortunately, valgrind itself must be suid bit for this to work, making this solution impractical.
* Compile binaries for an architecture with tagged memory [https://en.wikipedia.org/wiki/Tagged_pointer](https://en.wikipedia.org/wiki/Tagged_pointer), 
and interpret the result, for example, using QEMU. This solution has exactly the same problems as valgrind.
* Compilation in WASM. The most promising solution, given the availability of a large number of simple, interpretable, embeddable runtimes - 
[https://github.com/appcypher/awesome-wasm-runtimes](https://github.com/appcypher/awesome-wasm-runtimes):<br>
[https://github.com/wasmx/fizzy](https://github.com/wasmx/fizzy);<br>
[https://github.com/wasm3/wasm3](https://github.com/wasm3/wasm3).

The idea is quite simple - compile C into some interpretable virtual machine with a simple embeddable interpreter that is safe enough to have suid bit when executed.

It's not as hyped as rewriting in Rust, but the result can be obtained "tomorrow" if there is a desire!
