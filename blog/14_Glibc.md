## Ranting on glibc
<sup> December 16, 2022 </sup>

#glibc #rant

**Friday evening rant**

There is a library called glibc, whose authors are known for wanting to make everything "better" than others.<br>
The problem is that their solutions often turn out to be just "different", and then it's like - "Fuck, who came up with this and why".

There is such a function - pthread_cancel().

Do not use it anywhere and ever. Its behavior is as poorly defined as possible, and, in real life, it will lead you into some kind of deadlock. Because you won’t write tests for this section of code anyway.

By standard, all it has to do is call the specially registered cleanup handlers and end the thread. But colleagues from glibc decided that this was not enough for them! And they made the next feature - they call stack unwinder to destroy local objects on the stack in this thread.

How is it done?

* We load in runtime unwinder from gcc. No, it's not necessarily the same unwinder used by the rest of the program, it can be statically linked. [https://codebrowser.dev/glibc/glibc/nptl/pthread_cancel.c.html#99](https://codebrowser.dev/glibc/glibc/nptl/pthread_cancel.c.html#99)

* We throw an exception unknown to science. [https://codebrowser.dev/glibc/glibc/nptl/unwind.c.html#130](https://codebrowser.dev/glibc/glibc/nptl/unwind.c.html#130)<br>
Question for experts  - how does this exception interact with C++ exceptions? What about catch (...) ?

* I did not find the code that catches it on the top of the stack, but before it was also very intricate. Normal people would make one source with a .cxx extension, and catch it through the normal exception mechanism, but that's not the glibc developers way.

In fact, if your program brings its own unwind runtime with it, expect an arbitrary crash as a result of interacting with glibc/pthread_cancel.
