## About the security model
<sup> February 25, 2022 </sup>

#security

I promised to write about the security model here.

I’ll make a reservation right away, I’m writing about the security model of a personal laptop or desktop computer, the reasoning below does not apply to servers or even to your phone. It is also not applicable for any kind of corporate and BYOD laptops.
* It is convenient to consider this model as a two-user one: root + user, the fact that some system processes are running by nobody, actually doesn't change anything.
* All valuable data on your laptop (credit cards, passwords, access keys etc.) belongs to your personal account, root owns only system files and daemons.
* Therefore, if malware gains access to your user, then it gains access to all useful content on your computer. Getting root rights is NOT the target of the attack. Root on such a system is not needed for anything, not even needed to run a regular background process, because in any modern OS there are session user process supervisors. Let's say dbus.
* Hide your presence - is also not about root, BTW, these are slightly different holes in the system.

Hence an interesting consequence:

It is convenient to think of root not as a user who can do anything, but as a way to separate dangerous operations that require confirmation and all others. That is, sudo in such a system is not an escalation of privileges, but a signature that you run the current command, understanding what it does. Well, and, accordingly, a guarantee that the usual rm -rf $STEAM_ROOT/ will not demolish the entire root [https://github.com/ValveSoftware/steam-for-linux/issues/3671](https://github.com/ValveSoftware/steam-for-linux/issues/3671) (yes, I know that now rm - rf / works a little differently).

Therefore, I have a rather disregard for all sorts of CVE for #system0. It's cool, of course, that there is some kind of passage that allows attackers to get the root locally, but I hope that I was able to explain that sometimes it’s neither hot nor cold from this.

A separate issue - where the security perimeter in your laptop lies then. The answer is quite obvious - it lies in your browser. You download software from stores, you don’t run any shit from the Internet. The perimeter is in the sandboxing of the code running in the browser. In the processing of images by the browser. And so on. The browser must be loved, groomed, updated. In my opinion, the browser is the only (well, okay, sshd also) program in the system that makes sense to lick from a security point of view - all sorts of libc hardening, ASLR (pic), etc.

And certainly not in your /etc.

Why is this not about the phone? Because your phone is not a two-user environment, but a multi-user one, the users don't trust each other.

Why not about BYOD? Because in BYOD there are 3 users: you, root, and comrade major (who needs to both hide his presence for malware and detect it). Also an interesting model, but some other time.

A two-user computer with antivirus, by the way, is also a slightly different model. I do not use antivirus, I consider it a thing, on average (the medicine is worse than the disease), harmful.
