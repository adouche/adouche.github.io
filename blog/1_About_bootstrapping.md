## About bootstrapping
<sup> October 2, 2021 </sup>

#bootstrapping

I am very interested in build systems in all their various forms. So much so that I have my own build system for C++ code and an OSS packages build system, for both Darwin and Linux. For the latter, it's not just a package system, but a full-fledged distribution [stal/IX](https://github.com/stal-ix). 

I will talk more about these projects later, especially the second one. For now, I mention them as evidence of my great interest in build systems, distribution, and bootstrapping.

Today, I want to talk about the task of bootstrapping.

What is it? Imagine you are stranded on a deserted island with an empty computer (no OS), and a CD with program source code. You need to turn this into a working OS so that you won't be bored on the island (you didn't have a stewardess to keep you company).

This task has the following aspects:

1. Historical. How did we get from computers with punch cards to modern workstations and tablets?
2. Strictly practical. How to deploy a distribution with the smallest possible binary seed. Here is a very interesting study on how to go from a minimal 200-instruction hypervisor to a modern Linux distribution: [https://github.com/fosslinux/live-bootstrap/blob/master/parts.rst](https://github.com/fosslinux/live-bootstrap/blob/master/parts.rst).
3. Security. A reproducible build with a minimal binary seed is needed to ensure that there are no backdoors in the software. For example, the infamous Thompson attack: [https://www.cs.cmu.edu/~rdriley/487/papers/Thompson_1984_ReflectionsonTrustingTrust.pdf](https://www.cs.cmu.edu/~rdriley/487/papers/Thompson_1984_ReflectionsonTrustingTrust.pdf).<br>
A very funny option - [https://www.quora.com/What-is-a-coders-worst-nightmare/answer/Mick-Stute?share=1](https://www.quora.com/What-is-a-coders-worst-nightmare/answer/Mick-Stute?share=1).<br>
It's very valuable to know that your Alacritty build is sourced from GitHub and doesn't contain any malware or other malicious code. I try not to keep software that cannot be traced back to its source code from the ground truth.

There are several communities on the internet that are also concerned about this task:

1. [https://bootstrappable.org/](https://bootstrappable.org/)
2. [https://guix.gnu.org/](https://guix.gnu.org/) (very into bootstrapping)
3. [https://nixos.org/](https://nixos.org/) (to a lesser extent, they don't hesitate to use binary Rust builds, for example)

Colleagues from Guix even developed a couple of specialized languages for initial bootstrapping [https://www.gnu.org/software/mes/](https://www.gnu.org/software/mes/), [https://guix.gnu.org/blog/2019/guix-reduces-bootstrap-seed-by-50/](https://guix.gnu.org/blog/2019/guix-reduces-bootstrap-seed-by-50/). However, Guix has its own quirks, so I can’t recommend it for everyday use. Nix is more interesting in this regard. I used it as a package manager for any Linux, Darwin until I completed the task of bootstrapping my own distribution.

A few more things to note about bootstrapping:

* Not all languages can be bootstrapped. This means that some languages require a large initial blob of unknown quality. I try not to use such languages [https://elephly.net/posts/2017-01-09-bootstrapping-haskell-part-1.html](https://elephly.net/posts/2017-01-09-bootstrapping-haskell-part-1.html).

* Java was recently bootstrapped [https://bootstrappable.org/projects/jvm-languages.html](https://bootstrappable.org/projects/jvm-languages.html), but its build system (Gradle) is still not.

* Rust, in my opinion, is bootstrappable with great reserve:

1. To build version n, you need version n-1. This greatly increases the length of the bootstrap chain. About 20 builds currently?
2. The sources of the first versions in Standard ML are located unknown where. Bootstrapping is only possible using a third-party product [https://github.com/thepowersgang/mrustc](https://github.com/thepowersgang/mrustc).
3. It's not yet possible to repeat the chain on Apple M1.

* Go's bootstrapping is all good. The authors maintain version 1.4, the last one in C. With it, you can build a modern Go. There is also gccgo.

* Lisp is also very good. For example, I was able to extend the chain from interpreted ECL to compiled SBCL on Apple M1 in an hour.
