# Contributing

## Ground rules

Keep changes scoped and minimal. This is a fork of
[vtrlx/LuaGObject](https://github.com/vtrlx/LuaGObject); do not reformat or
rewrite upstream code that is not part of the change. Fork-only fixes are
welcome here.

Do not send fork-only changes upstream. Upstream has its own contribution
policy, including a ban on machine-generated contributions.

## Development setup

```bash
make LUA_VERSION=5.5                     # build lua_gobject_core.so
ninja -C build && ninja -C build test    # Meson build and test suite
```

Inside IgnisLua the built module is installed to
`deps/install/lib/lua/5.5/LuaGObject/`; the launcher picks it up from there.

## Code style

- Match the surrounding C style; `-Wall -Wextra` must stay clean.
- Reference counting changes must state, in the PR, which reference is taken or
  released and why. GC and finalization changes need a reproduction case.
- No narration comments: comment the reason, not the mechanics.

## Reporting bugs

Include Lua version, girepository/GLib version, build method, a minimal
reproduction, and whether the bug reproduces on upstream. Crashes are much
easier to fix with a backtrace.

## Pull requests

Explain the reference-counting or marshalling semantics you rely on, list the
tests run, and keep unrelated changes out of the diff.
