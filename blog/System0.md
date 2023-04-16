## About System0
<sup> January 31, 2022 </sup>

#system0

I think you already understood that #system0 is "my beauty":)

In it, I intend, to the best of my ability, to correct various problems that I encountered while interacted with Linux/Unix.

One such problem is the Unix property that the init inherits processes whose parent has died. I think that this is trash, waste, and sodomy (and one of the reasons for the emergence of cgroups, by the way). I'm pleased to see a completely supervised tree, if you know what I mean. This is convenient for both understanding and debugging. Long-lived processes need to be started using the session supervisor’s spawn command.

Therefore, I have [https://github.com/stal-ix/ix/blob/main/pkgs/bin/sched/staleprocs/staleprocs.sh](https://github.com/stal-ix/ix/blob/main/pkgs/bin/sched/staleprocs/staleprocs.sh) - a daemon that kills "stray" processes. If it flies, then I, of course, will add it to my init.

By the way, killing via SIGINT is there solely because if sway is killed via SIGKILL, then it regularly hangs the system tightly. Here is such a cool graphics stack in Linux.

Unfortunately, the same sway for some reason starts all processes through double fork.<br> 
Luckily, the fix is 3 lines of code.
