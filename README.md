# pgbcomp

Pokemon Game Boy compilation — six Gen 1 + Gen 2 carts in one launcher
binary, built on [GB-Recomp/gb-recompiled](https://github.com/GB-Recomp/gb-recompiled).

Bundles:

- [pokered](https://github.com/GB-Recomp/pokered) — Pokemon Red
- [pokeblue](https://github.com/GB-Recomp/pokeblue) — Pokemon Blue
- [pokeyellow](https://github.com/GB-Recomp/pokeyellow) — Pokemon Yellow
- [pokegold](https://github.com/GB-Recomp/pokegold) — Pokemon Gold
- [pokesilver](https://github.com/GB-Recomp/pokesilver) — Pokemon Silver
- [pokecrystal](https://github.com/GB-Recomp/pokecrystal) — Pokemon Crystal

Each cart is its own repo and can also be built standalone — this repo
is the umbrella that links them together with a multi-game picker.

## Build

You need a C/C++ compiler, CMake 3.16+, SDL2, libcurl, and OpenGL ES 2.
CMake pulls every dependency via `FetchContent` (no submodules):

```sh
mkdir build && cd build
cmake ..
cmake --build . -j$(nproc)
```

First configure takes a minute or two as it clones the 7 dependent
repos. Subsequent builds are incremental.

This produces a `pokegb` executable.

## Run

```sh
mkdir -p roms
cp /path/to/pokered.gb       roms/pokered.gb
cp /path/to/pokeblue.gb      roms/pokeblue.gb
cp /path/to/pokeyellow.gbc   roms/pokeyellow.gbc
cp /path/to/pokegold.gbc     roms/pokegold.gbc
cp /path/to/pokesilver.gbc   roms/pokesilver.gbc
cp /path/to/pokecrystal.gbc  roms/pokecrystal.gbc
./pokegb
```

The launcher's graphical picker lists every game; missing ROMs are
greyed out. After the first boot of a given cart, assets are extracted
into `assets/<id>/` and the ROM file is no longer needed.

You can also skip the picker via `./pokegb --game pokecrystal`.

In-game: Esc opens the runtime menu (palette / shader / SGB / borders /
savestates / Return to Launcher / Quit). On Gen 2 carts you'll also see
a **Mystery Gift** subsection — pick an item or decoration, click Send,
then trigger Mystery Gift from the cart's title menu to receive it.

## Customizing which version of a cart to use

Each game repo is pinned to its `main` branch by default. To pin
something different (a specific commit, a tag, a branch):

```sh
cmake -DPOKECRYSTAL_REF=<sha-or-tag-or-branch> ..
```

Same knob exists for `GBRT_REF`, `POKERED_REF`, `POKEBLUE_REF`,
`POKEYELLOW_REF`, `POKEGOLD_REF`, `POKESILVER_REF`.

## Adding more carts

Want to extend this compilation with another GB recomp? Add a
`FetchContent_Declare` for the new repo above the
`FetchContent_MakeAvailable` line in `CMakeLists.txt`, link its
`<id>_cart` target into the `pokegb` executable, and add an entry to
`g_games[]` in `launcher_main.cpp`.

Or — if it's a different series — start a parallel `<seriesname>comp`
umbrella that does the same thing for those carts.
