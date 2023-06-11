## Ranting on glibc
<sup> December 16, 2022 </sup>

#glibc #rant

**Friday evening rant**

There is a library called glibc, whose authors are known for wanting to make everything "better" than others.<br>
The problem is that their solutions often turn out to be just "different", and then it's like - "Fuck, who came up with this and why".

There is such a function - pthread_cancel().

Do not use it, anywhere, ever. Its behavior is poorly defined and in real life, it will lead you to some kind of deadlock. Because you won’t write tests for this section of code anyway.

According to the standard, all it should do is call specially registered cleanup handlers and terminate the thread. But colleagues from glibc decided that this was not enough! And they added the following feature - they call the stack unwinder to destroy local objects on the stack in this thread.

How is it done?

* We load the unwinder from gcc at runtime. No, it doesn't have to be the same unwinder that the rest of the program uses, it can be statically linked [https://codebrowser.dev/glibc/glibc/nptl/pthread_cancel.c.html#99](https://codebrowser.dev/glibc/glibc/nptl/pthread_cancel.c.html#99).

* We throw an exception unknown to science [https://codebrowser.dev/glibc/glibc/nptl/unwind.c.html#130](https://codebrowser.dev/glibc/glibc/nptl/unwind.c.html#130).<br>
Question for experts  - how does this exception interact with C++ exceptions? And with catch (...)?

* I did not find the code that catches it at the top of the stack, but it used to be quite intricate. Normal people would create one source file with the extension .cxx and catch it through the usual exception mechanism, but that's not the path of glibc developers.

So, if your program carries its own unwind runtime, expect an arbitrary crash as a result of interaction with glibc/pthread_cancel.
