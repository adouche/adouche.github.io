## Suffering by icons
<sup> February 04, 2022 </sup>

#svg #icons #rendering

With icons I suffered, of course:

* Paths to default icons in toolkits;
* What icons should be;
* Some of the icons in Adwaita are in png, some are in svg, these sets partially intersect. What this or that application will choose, one (idk who) knows. B&W icons are in svg, colored icons are in png;
* Svg loading is arranged through a plugin for gdk-pixbuf. Plugins for it are arranged in the most unnatural way of all known to me (I will try to write about how perversely developers arrange the loading of their plugins). Unfortunately, the latest version of this plugin, not rewritten in Rust, renders Adwaita's latest icons not very correctly;
* Debugging "what's actually loading" with strace doesn't really help, because every icon folder needs to contain an index.

That’s how we live - for high-quality rendering in png inkscape is required, for low-quality on the fly - Rust.<br>
Unfortunately, the Rust compiler doesn't even run right now, because it basically can't be linked statically.

All hopes for [#mrustc](https://github.com/thepowersgang/mrustc). Dude is getting closer and closer to supporting 1.54. It is clear that without a borrow checker, but who needs it not in the development process? It has its own implementation of proc macro (for obvious reasons), which does not require loading .so into the compiler.

I have now settled on svg icons with not quite correct rendering, because I've already tired of licking Epiphany ([https://wiki.gnome.org/Apps/Web](https://wiki.gnome.org/Apps/Web)). We'll have to see what's in store for the Chromium build path.

By the way, I almost won tearing in Epiphany. To do this, I built a webkit web process (which deals directly with rendering) on gtk4, in which everything is better with support for opengl / vulkan canvas, and the browser itself with gtk3. Because Igalia has already ported WebKIT to gkt4, but the browser itself has not yet. I think Igalia would choke on something for joy, knowing that I do this.
