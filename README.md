# image-chor

The image editor for [Petrichor64](https://github.com/Auxnon/petrichor64): the
**Ctrl+P** paint overlay. Over any running app that hasn't set
`attr{lock=true}`, it pauses the app, lets you click a mesh, sprite or tile, and
paint its texture in place, with a tool panel floating beside it in 3D and a
zoomed pixel canvas mode.

- **Keys:** B pen, F fill, I pick colour, 1-3 brush size, Z undo, C canvas,
  N new image, Esc steps back (canvas, then focus, then close), Ctrl+P closes.
- **SAVE** (lower right, once something is painted) writes the edits back into
  the app's image files after a confirm dialog. A packed game is read only.
- **NEW** gives the focused thing a fresh image of its own (16×16, or the
  sheet's tile size), unwrapping a mesh onto it. Saving writes
  `assets/<name>.png`, or `assets/<name>.glb` for a mesh.

## How it gets into chor

Petrichor's `studio` feature (on by default) fetches the latest `main` of this
repo when the engine is built and compiles `scripts/*.lua` into `chor`. An
overlay gets the privileged `app.*` table, so its code always comes from the
engine binary, never from the app being edited.

## Working on it

Run it from disk over a game, no rebuild needed:

```sh
chor path/to/game --overlay ~/projects/image-chor
```

To build `chor` with your local checkout instead of the latest on GitHub, set
`IMAGE_CHOR_DIR`:

```sh
IMAGE_CHOR_DIR=~/projects/image-chor cargo build --release
```

`scripts/main.lua` is the entry point; the header comment of each script says
what it owns. The scripts are written in the restricted style Petrichor's
AGENTS.md asks for (silt): calls hoisted out of argument lists, no anonymous
functions, no table deletion. The engine-side API is documented in Petrichor's
`guide/` (`pick.md`, `save.md`, `source.md`, `info.md`, `texinfo.md`,
`newtex.md`, `unwrap.md`, ...).
