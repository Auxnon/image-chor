# AGENTS.md — image-chor

image-chor is Petrichor64's image editor, written in Lua as a Petrichor app.
Today it runs as an **overlay**: Ctrl+P brings it up over a running game. A
**standalone** mode, where `chor` opens it on its own, is planned (below).

The engine is [Petrichor64](https://github.com/Auxnon/petrichor64), locally
`~/projects/petrichor64`. Its AGENTS.md is the reference for writing Lua for
the engine, testing without a human, and the overlay `app.*` API. Read it
before changing anything here; this file only covers what's specific to the
editor.

## How it ships

- Petrichor's `studio` feature (on by default) makes its build.rs fetch this
  repo's latest `main` and compile `scripts/*.lua` into `chor`
  (petrichor `src/builtin.rs`). Every file in `scripts/` is included, so a new
  script needs no change on the engine side.
- **`main` is what users get.** The next petrichor build anywhere pulls it, so
  never push a broken `main`; use a branch and `IMAGE_CHOR_REF` to try work
  out.
- Engine first: if a change here needs a new engine function (a new `app.*`
  call, say), land that in petrichor before pushing the script that calls it.
  Otherwise a petrichor build without it ships an editor that errors.
- `chor --version` prints the image-chor commit built in.

## Layout

```
scripts/main.lua     entry: shared state tables (P, F, U, H, INFO), main/loop/draw, keys
scripts/scene.lua    scene mode: hover, focus, painting straight onto the app's meshes
scripts/canvas.lua   canvas mode: the focused texture blown up flat on the gui
scripts/paint.lua    the tools (pen, fill, colour pick), undo
scripts/pal.lua      the palette and the overlay's own textures
scripts/panel.lua    the tool panel, one 128x128 image (billboard in scene, flat in canvas)
scripts/save.lua     the dithered frame, the SAVE button and its confirm dialog
scripts/newtex.lua   the NEW modal: a fresh image for the focused thing, unwrapped onto a mesh
icon.png             the app icon (assets/icon.png is the same image)
```

Each script's header comment says what it owns. Keep them that way.

## The overlay (what exists)

Ctrl+P over any app that hasn't set `attr{lock=true}` pauses it and loads this
editor as an overlay with the privileged `app.*` table:

- **Scene mode:** `app.pick()` finds the mesh, sprite or tile under the
  pointer, down to the texture and texel. Clicking focuses it, and a tool panel
  floats in 3D beside it. Strokes edit a copy of the texture (`gimg`) and push
  it back with `tex()`, so every mesh using it updates live.
- **Canvas mode** (C or ZOOM): the same texture drawn big and flat, same tools.
- **SAVE:** `app.source`/`app.save` write edits back into the app's image
  files (a tile into its square of the sheet, an engine-written glb back into
  the glb). A packed game (`app.info().packed`) can't be saved: READ ONLY.
- **NEW:** `app.newtex` gives the focused entity a fresh image (16x16, or the
  sheet's tile size from `app.texinfo`), unwrapping a mesh onto it.

Engine docs for these: petrichor `guide/pick.md`, `pause.md`, `view.md`,
`colors.md`, `project.md`, `close.md`, `source.md`, `save.md`, `info.md`,
`texinfo.md`, `newtex.md`, `unwrap.md`.

## Standalone (planned, not built)

The idea: open the editor on its own with a `chor` subcommand, with no game
running, e.g. `chor edit path/to/image.png`. The command's name isn't decided
(`edit` is a placeholder). Expected shape, to be settled when it's built:

- The engine loads this same code as a privileged app with no edit target,
  and tells it which mode it's in (an `app.info()` field, say).
- The editor starts in canvas mode on the given image (or a new blank one),
  with no scene mode or picking, since there's nothing to pick from.
- Saving writes straight to that file, and NEW makes a new file.
- Scene, chrome and canvas code is shared between the two modes; anything that
  assumes a running app (`app.pick`, `app.pause`, `app.view`) has to be behind
  the mode check.

When this lands, update this section, README.md, and petrichor's AGENTS.md and
`chor --help`.

## Working on it

- Run it from disk over a game, no rebuild:
  `chor path/to/game --overlay ~/projects/image-chor`.
- Build `chor` with this checkout instead of GitHub's `main`:
  `IMAGE_CHOR_DIR=~/projects/image-chor cargo build --release` in petrichor.
- Write Lua in the silt style petrichor's AGENTS.md describes: calls hoisted
  out of argument lists, no anonymous functions, no table deletion.
- Testing: petrichor AGENTS.md, *Testing an overlay*. Copy this repo to a
  scratch dir, add a `zz_test.lua` defining `TEST_HOOK`, aim `app.pick` at
  fixed screen points, and capture frames with `CHOR_SNAP`. The overlay hook's
  `dt` is nil, so step with `1/60`.
