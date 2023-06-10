## Tired with ~~all these~~ temporary files
<sup> November 21, 2022 </sup>

#temporaryfiles #stal/ix

Today I have a joke for you about my sense of beauty.

Personally, I hate programs that want to create temporary files. In general, "temporary file" is some nonsense, because why write ephemeral data to a persistent file?

Data can be buffered in memory or written through a pipe to another program’s input, or something similarly reasonable.

Usually, programs write something to temporary files as a hack, when some block of code already accepts int fd, and rewriting it to a normal interface is too lazy, or when some program in the chain does not know how to pipe.

In the first case, by the way, in modern Linux you can use memfd_create(), which is quite useful in terms of "sweeping garbage under the bed".

Another problem with temporary files is that various strange programs do not respect the TMPDIR setting and try to write directly to /tmp. Moreover, they come up with some wild ways of access differentiation and "non-intersection" with other instances.

Here is the contents of my session tmp:

```
./epiphany-pg-aaNPpb
./d9ca075b14fe84b587843f702e0d2466-
     {87A94AB0-E370-4cde-98D3-ACC110C5967D}
./6b047d326310867001cb39f5218a36ba-
     {87A94AB0-E370-4cde-98D3-ACC110C5967D}
./mc-pg
./mc-pg/mc-pg
./sway-ipc.10000.9709.sock
./wayland-1
./wayland-1.lock
./dbus.sock
./dbus-1
./dbus-1/services
./dbus.cfg
./ssh-XXXXXXJIAMlN
./ssh-XXXXXXJIAMlN/agent.9704
```

How many different ways of “uniqueness” did you count here?

I repeat, this infuriates me greatly, and I have set a goal for myself - all the programs that I use should be able to use TMPDIR.

By the way, a little off topic, TMPDIR should of course be a "complex" path, including user ID and session ID (for example, to effectively clean up this junk at the end of the session):

```
pg->echo ${TMPDIR}
/var/tmp/10000/10953
```

That’s why I don’t have a root /tmp folder in [stal/IX](https://github.com/stal-ix).

Why?

Because, by a strange coincidence, all programs that have their own strong opinions about the naming of the folder with temporary files try to write to /tmp.

Therefore, all such programs break, and I fix them all.<br>
To make this process not painful, I created a small library for this - [https://github.com/stal-ix/ix/tree/main/pkgs/lib/shim/ix](https://github.com/stal-ix/ix/tree/main/pkgs/lib/shim/ix).<br>
Its application is almost automated - [https://github.com/stal-ix/ix/blob/main/pkgs/bin/got/ix/ix.sh#L10](https://github.com/stal-ix/ix/blob/main/pkgs/bin/got/ix/ix.sh#L10).
