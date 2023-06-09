## The benefits of a reboot
<sup> March 20, 2023 </sup>

#reboot #heisenbug #system #entropy

A good, but very long text ([https://keunwoo.com/notes/rebooting/](https://keunwoo.com/notes/rebooting/)) that contains two simple ideas:
1. Entropy accumulates in any system. In other words, there is heisenbug in the system [https://ru.wikipedia.org/wiki/Heisenbug](https://ru.wikipedia.org/wiki/Heisenbug).
2. Rebooting (VM, host, program) is a simple way to return the system from a state with accumulated error to a state without such error.

People who know me better know that I like to give a couple of strange architectural tips:
* Periodically restart your programs on the cluster. Usually, updating versions is enough if it is regular.
* A slightly stranger tip - insert abort() into your program in random places. In prod, specifically in prod.

Before reading the following example from my practice, try to imagine why this is good and right, and what problems it will protect you from.

My unbound hung. Well, it hung and hung, somewhere in select/poll/etc, did not respond to requests and did nothing itself.

Of course, you can tilt at windmills, send a completely meaningless stack trace (because without symbols) in the upstream, and wait for the sun to shine.

I did exactly what I suggested above - started regularly reducing the entropy of the system by regularly restarting unbound -
[https://github.com/pg83/ix/blob/main/pkgs/bin/unbound/runit/ix.sh#L8](https://github.com/pg83/ix/blob/main/pkgs/bin/unbound/runit/ix.sh#L8), with using `/bin/timeout 300`.

Once every 5 minutes is quite enough for the DNS cache to be useful.
