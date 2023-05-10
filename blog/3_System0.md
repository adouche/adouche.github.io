## About System0
<sup> January 31, 2022 </sup>

#system0

I think you already understand that system0 is "my precious" :)

In it, I intend to fix various problems that I have encountered while interacting with Linux/Unix to the best of my ability.

One such problem is the Unix property of inheriting processes by init when their parent dies. I believe that this is trash, hysteria, and sodomy (and one of the reasons for the emergence of cgroups, by the way). I like to see a completely supervised tree, if you know what I mean. It is convenient for understanding and debugging. Long-lived processes should be launched through the command spawn of the session supervisor.

That's why I have [https://github.com/stal-ix/ix/blob/main/pkgs/bin/sched/staleprocs/staleprocs.sh](https://github.com/stal-ix/ix/blob/main/pkgs/bin/sched/staleprocs/staleprocs.sh) - a daemon that kills "stray" processes. If this works well, then I will certainly add it to my init.

By the way, killing via SIGINT is there solely because if sway is killed via SIGKILL, it regularly hangs the system completely. That's such a cool graphics stack in Linux.

Unfortunately, sway for some reason launches all processes through double fork.<br>
Fortunately, the fix is just 3 lines of code.
