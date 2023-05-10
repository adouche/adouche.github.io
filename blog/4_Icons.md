## Struggled with icons
<sup> February 04, 2022 </sup>

#svg #icons #rendering #mrustc

With icons, I certainly struggled:

* Paths to default icons in toolkits;
* Which icons should be;
* Which icons should be used;
* Some icons in Adwaita are in png, some are in svg, and these sets partially overlap. Only one (idk who) knows which application will choose which. Black and white icons are in svg, and colored ones are in png;
* Loading svg is done through a plugin for gdk-pixbuf. Plugins for it are organized in the most unnatural way of all that I know (I will try to write about how developers pervert the loading of their plugins). Unfortunately, the latest version of this plugin, not rewritten in Rust, does not render the latest Adwaita icons very correctly;
* Debugging "what is actually being loaded" through strace doesn't help much because each folder with icons must contain an index.

That’s how we live - for high-quality rendering in png, inkscape is required, for low-quality on the fly - Rust.<br>
Unfortunately, my Rust compiler doesn't even run right now because it can't be linked statically on principle.

All hope is on [#mrustc](https://github.com/thepowersgang/mrustc), the dude is getting closer and closer to supporting 1.54. Of course, without a borrow checker, but who needs it outside of development? He has his own implementation of proc macro (for obvious reasons), which does not require loading .so into the compiler.

Right now I've stopped at svg icons with not quite correct rendering because I'm tired of licking Epiphany ([https://wiki.gnome.org/Apps/Web](https://wiki.gnome.org/Apps/Web)). I need to see what awaits in building Chromium.

By the way, I almost defeated tearing in Epiphany. To do this, I built the webkit web process (which is responsible for rendering) on gtk4, which is better with support for opengl/vulkan canvas, and the browser itself on gtk3. Because Igalia has already ported WebKIT to gkt4, but the browser itself has not yet. I think Igalia would be overjoyed to hear that I was doing this.
