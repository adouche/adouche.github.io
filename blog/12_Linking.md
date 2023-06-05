## Static VS dynamic linking
<sup> November 02, 2022 </sup>

#static_linking #dynamic_linking #gtk #wayland

In the "static vs dynamic linking" battle, there is one factor that no one really talks about, or maybe I just haven't come across it before.

It’s aesthetics!

Moreover, it is not about the aesthetics of simplicity, when in the case of static linking, all sorts of .so files are not scattered throughout the file system, which, without a package manager, it is unclear where they came from and what they are needed for. This is obvious and not very interesting.

In the post about [Mesa](8_Mesa.md), I briefly mentioned how it is built.

It was about the fact that, during the process of linking 3 .so files, 10 .a files are built and combined in an arbitrary manner into the corresponding .so files using a linker script to hide common parts.

Why is it done this way? Why not just make 10 .so files that link to each other as they should, without all this code duplication magic and all that?

Dynamic linking aesthetics require this!

Because it’s not acceptable to dump a bunch of incomprehensible .so files on the user without any clear scope, that do who knows what. It is necessary to give the user a clear set of artifacts with a clear scope, but the fact that the code is duplicated between them - who cares?

By the way, my recent story about symbol duplication in GTK and Wayland libraries can be attributed to exactly the same topic. It's insane to isolate these three unfortunate functions into some completely artificial library, how to sell it to the user, how to make it a package in distributions, and other unpleasant questions.

Conversely, static linking stimulates the emergence of a greater quantity of more granular dependencies because these are artifacts of the build time only and are not visible to the user. Therefore, they can be arbitrarily shitty, even with just one function in each if it is convenient. No one will see it live anyway.

*(An example of such libraries splitting into parts - in the next series, as there is already a lot of text here.)*

Therefore, the internal aesthetics inherent in a particular linking method actually dictate how the code will be split into modules, and (in my personal opinion) static linking leads to a better splitting, reflecting the code essence.
