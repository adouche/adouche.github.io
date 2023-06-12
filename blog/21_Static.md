## Static linking considered useful
<sup> April 27, 2023 </sup>

#static_linking #stal/ix #security

I came up with a funny idea about security in the context of static linking, which I have never encountered before.<br> 
Let's take, for example, ROP [https://en.wikipedia.org/wiki/Return-oriented_programming](https://en.wikipedia.org/wiki/Return-oriented_programming).<br>
It is a construction that allows us to create malicious code that executes within the address space of a process using parts of the program itself.  

It is quite simple - we take all the suffixes of all possible functions and modify the stack so that each subsequent ret happens in the next piece of the program we need:
```shell
func1:
...
A
B
ret

func2:
...
C
D
ret
```

By augmenting the call stack, we can execute:
```shell
A
B
ret
C
D
ret
...
```

Now, the astute reader might ask - where do we get these pieces? Most often, we can find them in well-known and stable code sequences, such as libc.so.<br>
The number of reusable fragments is proportional to the total amount of executing code.

Therefore, oddly enough, static linking provides fewer building materials for attacks than when you link a bunch of dead code from a bunch of .so files (the total size of all .so files for almost any program is greater than the same program but linked statically). This is because linker discards unused functions.<br>
And so it is "more secure"!
