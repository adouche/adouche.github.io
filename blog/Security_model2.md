## The security model. Episode 2
<sup> March 08, 2022 </sup>

#security

Continuing the security model theme:

[https://www.opennet.ru/opennews/art.shtml?num=56818](https://www.opennet.ru/opennews/art.shtml?num=56818)<br>
[https://www.opennet.ru/opennews/art.shtml?num=56812](https://www.opennet.ru/opennews/art.shtml?num=56812)

Two more vulnerabilities that I consider unimportant. And some considerations about their implementation:
* All sorts of complex security models are badly "linked", and they turn out to be somehow badly interacting. Like selinux + cgroups.
* The more complex the system, the more difficult it is for a leatherbag to think about it.

Therefore, I believe that models like "everything is permitted / everything is prohibited" rule because of their simplicity - it is more difficult to mess up in their implementation, and it is easier to think about them. And, despite the fact that they cannot express all sorts of shitty complex policies, they turn out to be safer.

How to deal with inherent complexity? I believe that with the help of a hierarchy, the top level is divided into two "all / nothing" layers, within each layer you can run a container with the same simple division.
