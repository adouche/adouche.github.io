## Generating stable unique keys
<sup> December 21, 2022 </sup>

#bootstrap #ix

I solved a long-standing problem here - generating unique keys for various kinds of programs, but so that they are stable from generation to generation, that is, if you need to rebuild the package with the same input data, then the resulting keys should be the same.

Usage example:
```shell
# ix mut system bin/dropbear/runit --seed="dead beef"
```

When installing dropbear (this is such an alternative to ssh) it wants to generate host keys that should not change for about the entire lifetime of the installed system.

It uses random() for this, which does not correspond to the bright idea of "clean build" - the package is a pure function from its inputs (I remind you that this is an important condition for the ability to reuse RO packages between different users (not necessarily even on the same host)).

Actually, the solution to this problem is divided into 2 parts:

* Slightly rewrite the key generation utility so that it does not go to /dev/random, but to a pre-prepared file - [https://github.com/pg83/ix/blob/main/pkgs/bin/dropbear/runit/keygen/ix.sh#L7](https://github.com/pg83/ix/blob/main/pkgs/bin/dropbear/runit/keygen/ix.sh#L7).

* Submit this file to the host key generator and generate these keys. So we get a "clean" package, which depends only on its seed parameter - [https://github.com/pg83/ix/blob/main/pkgs/bin/dropbear/runit/hostkeys/ix.sh#L14](https://github.com/pg83/ix/blob/main/pkgs/bin/dropbear/runit/hostkeys/ix.sh#L14).

Thus, we have, as it were, a "random" set of keys, which is a pure function of seed. The task of the user of the system when setting up the machine is to specify this same seed so that it is unique (this can be done by the installer transparently for the user).

In the process, I got an interesting artifact - a "package with prepared entropy". This is how I ask that it be available at the time of preparing the keys - [https://github.com/pg83/ix/blob/main/pkgs/bin/dropbear/runit/hostkeys/ix.sh#L4](https://github.com/pg83/ix/blob/main/pkgs/bin/dropbear/runit/hostkeys/ix.sh#L4), and this is how its implementation looks [https://github.com/pg83/ix/blob/main/pkgs/aux/entropy/ix.sh](https://github.com/pg83/ix/blob/main/pkgs/aux/entropy/ix.sh),
[https://github.com/pg83/ix/blob/main/pkgs/aux/entropy/ix.sh#L10](https://github.com/pg83/ix/blob/main/pkgs/aux/entropy/ix.sh#L10) - here is all the pulp, we pass seed to openssl.

```
pg-> ./ix build aux/entropy 
  --entropy_seed="dead beef" 
  --entropy_size=100
READY /ix/store/7aP.../touch
pg-> ls -ls /ix/store/7aP.../share/entropy
     100 ... /ix/store/7aP.../share/entropy
```

The artifact is actually quite useful, because at system setup time there are quite a few processes that require a unique host ID to be generated, such as /etc/machine-id, which is used by dbus.

In general, it is clear that we do not worsen the process compared to how it was before - it was "we use the entropy at the current time as a seed to generate some files", it became - "we save the entropy at some point in time, and use it to generate the same files over and over." That is, we store not the result of the generating, but its seed.

By the way, I found beautiful while reading the source - [https://github.com/mkj/dropbear/blob/master/dbrandom.c#L270](https://github.com/mkj/dropbear/blob/master/dbrandom.c#L270).<br>
Here it is written how normal people build their initial entropy for work.

(Those who tried to blame me for the file [https://github.com/catboost/catboost/blob/master/util/random/entropy.cpp#L40](https://github.com/catboost/catboost/blob/master/util/random/entropy.cpp#L40) should be ashamed - everyone does it!).
