## About the security model
<sup> February 25, 2022 </sup>

#security #system0

I promised to write about the security model here.

I should note that I am writing about the security model of a personal laptop or desktop computer, and the following reasoning does not apply to servers or even your phone. It is also not applicable to any kind of corporate or BYOD laptops.

* It is convenient to consider this model as a two-user one: root + user. The fact that some system processes are run by nobody actually doesn't change anything.
* All valuable data on your laptop (credit cards, passwords, access keys etc.) belongs to your personal account, root only owns system files and daemons.
* Therefore, if malware gains access to your user, it gains access to all the useful content on your computer. Obtaining root privileges is NOT the goal of the attack. Root is not needed for anything in such a system, not even for running a regular background process, because any modern OS has session user process supervisors, such as dbus.
* Hiding your presence is also not about root, BTW, it's a slightly different system vulnerability.

Hence an interesting consequence.

It is convenient to consider root not as a user who can do everything, but as a way to separate dangerous operations that require confirmation from all others. That is, sudo in such a system is not privilege escalation, but a signature that you are running the current command understanding what it does. And, accordingly, a guarantee that an ordinary rm -rf $STEAM_ROOT/ will not destroy the entire root  [https://github.com/ValveSoftware/steam-for-linux/issues/3671](https://github.com/ValveSoftware/steam-for-linux/issues/3671) (yes, I know that now rm - rf / works a little differently).

Accordingly, I have a rather indifferent attitude towards various CVEs for system0. It's cool, of course, that there is some vulnerability that allows getting root locally, but I hope I have been able to explain that sometimes it doesn't matter.

A separate topic is where the security perimeter lies in your laptop. The answer is quite obvious - it lies in your browser. You download software from stores, don’t run any crap from the Internet. The perimeter is in the sandboxing the code running in the browser. In processing images by the browser. And so on. You have to love, cherish, and update your browser. The browser is the only (well, okay, also sshd) program in the system that I see the point of licking from a security point of view - all kinds of libc hardening, ASLR (pic), etc.

And definitely not in your /etc.

Why is this not about phones? Because your phone is not a two-user environment, but a multi-user one, where users don't trust each other.

Why not about BYOD? Because in BYOD there are 3 users - you, root, and Major Comrade (who needs to hide his presence from malware and detect it). It's also an interesting model, but maybe another time.

A two-user computer with antivirus, by the way, is also a slightly different model. I don’t use antivirus, I consider it harmful on average (the cure is worse than the disease).
