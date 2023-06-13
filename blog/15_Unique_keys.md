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

The artifact is actually quite useful, because at system setup time there are quite a few processes that require a unique host ID to be generated, such as /etc/machine-id, which is used by dbus.

In general, it is clear that we do not worsen the process compared to how it was before - it was "we use the entropy at the current time as a seed to generate some files", it became - "we save the entropy at some point in time, and use it to generate the same files over and over." That is, we store not the result of the generating, but its seed.

By the way, I found beautiful while reading the source - [https://github.com/mkj/dropbear/blob/master/dbrandom.c#L270](https://github.com/mkj/dropbear/blob/master/dbrandom.c#L270).<br>
Here it is written how normal people build their initial entropy for work.

(Those who tried to blame me for the file [https://github.com/catboost/catboost/blob/master/util/random/entropy.cpp#L40](https://github.com/catboost/catboost/blob/master/util/random/entropy.cpp#L40) should be ashamed - everyone does it!).
