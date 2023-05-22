## About the security model. Episode 2
<sup> March 08, 2022 </sup>

#security

Continuing the security model topic.

[https://www.opennet.ru/opennews/art.shtml?num=56818](https://www.opennet.ru/opennews/art.shtml?num=56818)<br>
[https://www.opennet.ru/opennews/art.shtml?num=56812](https://www.opennet.ru/opennews/art.shtml?num=56812)

Another two vulnerabilities that I consider unimportant. And some thoughts on their implementation:
1. Complex security models do not "compose" well, and end up being poorly interacting. Like selinux + cgroups;
2. The more complex the system, the harder it is to think about it for a leatherbag.

Therefore, I believe that "all or nothing" type models rule due to their simplicity - it is harder to mess up in their implementation, and they are easier to think about. And despite the fact that they cannot express various crap complex policies, they turn out to be safer.

How to deal with inherent complexity? I believe that through hierarchy - the top level is divided into two "all/nothing" layers. Within each layer you can run a container with the same simple division.
