# LuaGObject (sysv86 fork)

LuaGObject dynamically generates Lua bindings to libraries that support
GObject-Introspection — GLib, Gio, GTK 4, libadwaita, Pango and many more — plus
the Cairo bindings needed by parts of GTK.

This repository is a fork of [vtrlx/LuaGObject](https://github.com/vtrlx/LuaGObject),
maintained for [IgnisLua](https://github.com/sysv86/IgnisLua). Upstream remains
the canonical project; this fork exists because IgnisLua needs two things that
are not upstream yet.

Licensed under an [MIT-style](http://www.opensource.org/licenses/mit-license.php)
license — see `LICENSE`.

Tested with PUC-Rio Lua 5.1–5.5 and LuaJIT2. Other Lua implementations are not
officially supported. The project descends from [LGI](http://github.com/lgi-devs/lgi)
and shares much of its internals and structure.

## What this fork changes

1. **Lua 5.5 build support.** The core module is built against Lua 5.5 and is
   loaded by the IgnisLua launcher from `deps/install/lib/lua/5.5/LuaGObject/`.
2. **GC ownership hardening.** `object_gc` used to unref every object a proxy
   pointed at. For objects Lua never held a reference to — borrowed values and
   objects already finalized by GTK — that dereferenced stale memory and
   crashed inside `g_type_fundamental()`. The fork keeps a registry of objects
   Lua actually owns and only unrefs those; anything else is left alone.
3. **Closure and ParamSpec retention.** Signal closures created for Lua
   callbacks are anchored so a `GClosure` cannot be freed while the signal is
   still connected, and `GObject.ParamSpec` is pre-registered so a `pspec`
   arriving from C always has a valid repo table.

Everything else is upstream code. Diff against the `vtrlx/LuaGObject` master
branch to see the fork's changes in full.

## Building

GIRepository-2 >= 2.80.0 development files are required
(`libgirepository-2.0-dev` on Debian-based systems, `glib2-devel` on
RedHat-based systems).

### GNU Make

```bash
make LUA_VERSION=5.5
sudo make install LUA_VERSION=5.5 PREFIX=/usr
```

On BSD systems use `gmake`. Plain GNU Make does not detect a system Lua through
pkg-config; install PUC-Rio Lua from source if you build this way.

### Meson

```bash
meson setup build --prefix=/usr -Dlua-pc=... -Dlua-bin=...
ninja -C build
ninja -C build test
sudo ninja -C build install
```

Visual Studio 2013 and later are supported through Meson as well; see the
upstream README for the `%INCLUDE%`/`%LIB%`/`%PATH%` details.

### Inside IgnisLua

IgnisLua keeps a prebuilt copy of this fork under `deps/install/`, which the
launcher adds to `package.path`/`package.cpath` automatically. To rebuild it
in place:

```bash
make -C deps/LuaGObject/LuaGObject LUA_VERSION=5.5
cp deps/LuaGObject/LuaGObject/lua_gobject_core.so \
   deps/install/lib/lua/5.5/LuaGObject/lua_gobject_core.so
```

## Usage

```lua
local Gtk = require("LuaGObject").require("Gtk", "4.0")
local Gio = require("LuaGObject").require("Gio", "2.0")

local app = Gtk.Application({ application_id = "org.example.App" })
```

Documentation lives in `docs/`. Older LGI-era examples are in `samples/`; some
of them, especially the GTK ones, are dated.

### Migrating from LGI

Replace `require('lgi')` with `require('LuaGObject')`. For build systems and CI
pipelines, note that `corelgilua51.{so,dll}` was renamed to
`lua_gobject_core.{so,dll}`, and all `lgi/` paths became `LuaGObject/`.

## Contributing

Bug reports and pull requests for the fork belong in
[sysv86/LuaGObject](https://github.com/sysv86/LuaGObject); use the templates in
`.github/`. Do not send fork-only changes upstream — open them against this
repository instead.

Upstream `vtrlx/LuaGObject` has its own contribution policy, including a ban on
machine-generated contributions. Anything intended for upstream must satisfy
that policy; changes in this fork are not affected by it.

## Credits

LuaGObject exists because of Christian Hergert's initial port of LGI to
GIRepository 2.80, and because of the LGI developers: Pavel Holejsovsky, Uli
Schlachter, Jasper Lievisse Adriaanse, Ildar Mulyukov, Nils Nordman, Ignas
Anikevicius, Craig Barnes, Nicola Fontana, Andreas Stührk, Aaron Faanes,
Jiří Klimeš, Garrett Regier, Kenneth Zhou, Travis Hoppe, Tobias Jakobs,
Heiko Becker, Vincent Bermel and Szunti.

Release history up to 0.10.5 is documented in the upstream README; older
pre-fork versions are listed in LGI's history.
