## Static VS dynamic linking
<sup> November 02, 2022 </sup>

#staticlinking

In the "static vs dynamic linking" battle, there is one factor that no one really talks about, or I just haven't met before.

This is aesthetics!

Moreover, it’s not simple aesthetics, that, they say, in the case of static linking, all sorts of different .so are not lying around the entire fs, about which, without a package manager, it’s impossible to say where they come from and what they are for. It lies on the surface, and is not very interesting.

Once, when I was talking about #mesa, I casually mentioned how it is building.

It was about the fact that, in the process of building 3 .so’s, 10 .a’s are built, and they are somehow, rather arbitrary, combined into the corresponding .so files, using a linker script to hide common parts.

Why is this being done? Why not make 10 .so’s that just link to each other like they are supposed to, and there is no code duplication magic and all that?

This is what dynamic build aesthetics demand!

Because it’s not good to dump a bunch of incomprehensible .so on the user, without some kind of understandable scope, which do who knows what. It is necessary to give the user a clear set of artifacts, with a clear scope, but the fact that the code is duplicated between them - who cares?

By the way, my recent story about the duplication of symbols in the #gtk and #wayland libraries can be attributed exactly to the same topic. It's crazy to separate these 3 unfortunate functions into some completely artificial library, how to sell it to the user, how to package it in distributions, and other unpleasant questions.

Conversely, static linking encourages more and more granular dependencies, because these are purely build-time artifacts and are not visible to the user. Therefore, they can be arbitrarily shitty, at least 1 function in each, if it is so convenient, no one will see it live anyway.

*(an example of such a division of libraries into parts - in the next series, and here is already a lot of text).*

So the inherent aesthetics of a particular linking method actually dictate how the code will be split into modules, and (my personal opinion) static build leads to a better, reflective the code essence, separation.
