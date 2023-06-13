## Generating stable unique keys
<sup> December 21, 2022 </sup>

#bootstrap #ix

I solved a long-standing problem - generating unique keys for various programs, but in such a way that they are resistant from generation to generation, meaning that if the package needs to be rebuilt with the same input data, then the resulting keys should be the same.

Usage example:
```shell
# ix mut system bin/dropbear/runit --seed="dead beef"
```

When installing Dropbear (such an alternative to SSH), it wants to generate host keys that should not change for about the lifetime of the installed system.

It uses random() for this, which does not correspond to the bright idea of "clean build" - the package is a pure function of its inputs (as a reminder, this is an important condition for the ability to reuse RO packages between different users (not necessarily even on the same host)).

Actually, the solution to this problem was divided into two parts:

* Slightly modify the key generation utility to use a pre-prepared file instead of /dev/random - [https://github.com/pg83/ix/blob/main/pkgs/bin/dropbear/runit/keygen/ix.sh#L7](https://github.com/pg83/ix/blob/main/pkgs/bin/dropbear/runit/keygen/ix.sh#L7).

* Provide this file to the host key generator and generate these keys. This way, we get a "clean" package that depends only on its seed parameter - [https://github.com/pg83/ix/blob/main/pkgs/bin/dropbear/runit/hostkeys/ix.sh#L14](https://github.com/pg83/ix/blob/main/pkgs/bin/dropbear/runit/hostkeys/ix.sh#L14).

Thus, we have a kind of "random" set of keys, which is a pure function of seed. The task of the user of the system when setting up the machine is to specify this seed so that it is unique (this can be done by the installer transparently for the user).

During the process, I created an interesting artifact - a "package with prepared entropy". This is how I request that it be available at the time of keys preparation - [https://github.com/pg83/ix/blob/main/pkgs/bin/dropbear/runit/hostkeys/ix.sh#L4](https://github.com/pg83/ix/blob/main/pkgs/bin/dropbear/runit/hostkeys/ix.sh#L4), and this is what its implementation looks like - [https://github.com/pg83/ix/blob/main/pkgs/aux/entropy/ix.sh](https://github.com/pg83/ix/blob/main/pkgs/aux/entropy/ix.sh),
[https://github.com/pg83/ix/blob/main/pkgs/aux/entropy/ix.sh#L10](https://github.com/pg83/ix/blob/main/pkgs/aux/entropy/ix.sh#L10) - this is where all the pulp is, passing the seed to OpenSSL.

```
pg-> ./ix build aux/entropy 
  --entropy_seed="dead beef" 
  --entropy_size=100
READY /ix/store/7aP.../touch
pg-> ls -ls /ix/store/7aP.../share/entropy
     100 ... /ix/store/7aP.../share/entropy
```

This artifact is quite useful because during system installation there are many processes that require generating a unique host identifier, such as /etc/machine-id, which is used in D-Bus.

Overall, it is clear that are not making the process worse compared to how it was before - we used to "use entropy at the current moment in time as a seed to generate some files", and now we "save entropy at some point in time and use it to generate these same files again and again." That is, we store the seed, not the result of generation.

By the way, I found something beautiful while reading the source code - [https://github.com/mkj/dropbear/blob/master/dbrandom.c#L270](https://github.com/mkj/dropbear/blob/master/dbrandom.c#L270).<br>
Here it is written how normal people gather initial entropy for work.

(Those who tried to blame me for this file [https://github.com/catboost/catboost/blob/master/util/random/entropy.cpp#L40](https://github.com/catboost/catboost/blob/master/util/random/entropy.cpp#L40) should be ashamed - many do it this way!)
