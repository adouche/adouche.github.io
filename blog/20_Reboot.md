## The benefits of a reboot
<sup> March 20, 2023 </sup>

#reboot #heisenbug #system #entropy

[https://keunwoo.com/notes/rebooting/](https://keunwoo.com/notes/rebooting/) - A good, but very long text that contains two simple ideas:
1. Entropy accumulates in any system. In other words, there is heisenbug in the system [https://ru.wikipedia.org/wiki/Heisenbug](https://ru.wikipedia.org/wiki/Heisenbug).
2. Rebooting (VM, host, program) is a simple way to return the system from a state with accumulated error to a state without such error.

People who know me better know that I like to give a couple of strange architectural tips:
* Periodically restart your programs on the cluster. Version updates are usually sufficient if they are regular.
* A slightly weirder advice - to put abort() in random places in your program. In the prod, precisely in the prod.

Before reading the following example from my practice, try to imagine why this is good and right, and what problems it will protect you from.

My unbound is stuck. Well, it hung and hung, somewhere in select/poll/etc, did not respond to requests and did nothing itself.

You can, of course, tilt at windmills, send a senseless stack trace (because it has no symbols) into upstream, and wait for the sun to shine.

I did exactly what I suggested above - I began to regularly reduce the entropy of the system by regularly restarting unbound -
[https://github.com/pg83/ix/blob/main/pkgs/bin/unbound/runit/ix.sh#L8](https://github.com/pg83/ix/blob/main/pkgs/bin/unbound/runit/ix.sh#L8), with using `/bin/timeout 300`.

Once every 5 minutes is enough for dns cache to be useful.
