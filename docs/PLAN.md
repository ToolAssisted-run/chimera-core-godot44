# The Godot 4.4 core: plan and log

This core is [chimera-core-godot47](https://github.com/ToolAssisted-run/chimera-core-godot47)
with Godot 4.4.1 pinned in place of 4.7.2. Every decision about how a Godot
game becomes a Chimera machine is in that repository's docs/PLAN.md, and the
code (`module/chimera`, `waterbox/`, `tests/`) is the same files; a change
to one is made to every Godot core.

## Why a core per version (user-decided, 2026-10-20)

A game exported with Godot 4.3 or later carries its scripts compiled to a
token format that only the same version loads, and a pack made by a newer
Godot is refused by an older one. The owner chose cores for 4.4, 4.5 and 4.6
beside the 4.7 one, each its own package, repository and roster entry (one
Chimera package holds one engine). The 4.4 core also takes games of 4.0 to
4.3, for which there is no core.

## What differs from the Godot 4.7 core

- `module/chimera/chimera_compat.h`: before 4.7 the display server's enums
  are DisplayServer's own (4.7 moved them to DisplayServerEnums); before 4.6
  the display server's header is in `servers/`; 4.4 has no
  `GDSOFTCLASS`, which the compat header gives it.
- Godot 4.4 asks the OS for no working folder (`get_cwd` came in 4.6), and
  its file access has no access time or size by name, and its OS no
  text encodings (all came in 4.5); the module leaves those out.
- Godot's own build stops at linking its program before 4.6;
  `build-godot.sh` takes that as done. 4.4 leaves each library beside its
  sources instead of in `bin/`; `link-core.sh` looks in the whole build.
- This core also takes games of Godot 4.0 to 4.3 (there is no core for
  them); every other Godot core sends those here.
- The test games (`tests/*.pck`) are exported by the Godot 4.4.1 editor
  (`tests/make-test-game.sh`); a 4.7 pack does not load here.
- The pack's header says which Godot exported it, and a game of another
  version is refused with the name of the core to use (in every Godot core).

## Measured (2026-10-20)

- `waterbox/run-gate.sh`: 40 passed, 0 failed, on Dodge the Creeps and the
  probe exported by the Godot 4.4.1 editor: native = sandbox on every
  picture, the sound and the probe's 154 lines; a savestate before every
  frame (41,198,524 bytes) changes nothing.
- `tests/run-frontend.sh`: 5 ok (chimera-run with a movie file and a
  project, Chimera's window, the default bindings).

## Drawing Size (2026-10-28, chimera issue 380)

As the Godot 4.7 core's (its docs/PLAN.md): the Drawing Size setting (100%, 75%,
50% or 25%) gives the game a window of that share of the size it asks for,
and the game draws at that size, a quarter of the pixels at 50%. Godot draws
every pixel on the CPU here, so a game that stretches its picture to its
window runs about twice as fast or more; one with a fixed size of its own is
not made faster. It is part of the machine: a movie needs the same value.
The test script checks that at 50% Dodge the Creeps draws 240x360 and plays
the same steps, native and in the sandbox.

## Drawing Code: AVX2 where the computer has it (user-decided, 2026-10-28, chimera issue 380)

As the Godot 4.7 core's (its docs/PLAN.md): the Drawing Code setting, AVX2 by
default, lets llvmpipe make x86-64-v3 code (256-bit vectors) on a computer
that has all of it (mesa-patches/0002), the same on every such computer;
Baseline, and a computer without it, keep the baseline of patch 0001. The
game plays the same either way. The test script checks the 256-bit code in
the sandbox and natively, native = sandbox with it, and that Baseline still
makes 128-bit code and plays the same steps.


## A game that asks for a file of the player's own (user-decided, 2026-11-14, chimera issue 567)

Super Mario Bros. Remastered 1.1.0 (GPL-3.0, Godot 4.6) did not start in
the Godot 4.6 core, and it needs the player's own Super Mario Bros. NES ROM.
The owner asked for support only through changes every Godot game could use
(2026-11-14). Two were needed.

- **The files slot takes a file of any kind** (user-decided, 2026-11-14).
  The game looks in the folder of its executable for any file ending in
  `.nes`, checks its hash and copies it to `user://baserom.nes`. The core
  already puts the files of the project's "Other files beside the game"
  slot in that folder (`/game`), but the slot listed eight extensions, and
  Chimera refused a project with a `.nes` there ("file '...' is not a format
  slot 'files' accepts"). The slot now declares no formats
  (`"formats": []`): Chimera takes any file there (a rule it documents and
  tests), the file picker shows every file, and the core is unchanged.
- **user://logs is there at power-on** (user-decided, 2026-11-14: "No side
  effects in Chimera are allowable. Any file system modifications need to be
  fully emulated."). The game's mod loader opens `user://logs/modloader.log`
  for writing without checking the result. On a desktop, Godot's own log
  file has made that folder; here Godot's log goes nowhere (see "Save
  data"), the open failed, and the export template, which leaves out
  GDScript's null checks, crashed the core at start ("the core crashed: it
  read or ran address (nil)", in `MethodBindTR<bool, String const&>`). The
  core's disk now holds an empty `/user/logs` from power-on. That disk is
  the core's memory, so the folder is in every state and nothing is written
  to the computer's files. An empty folder is no save data.

Tests: `waterbox/run-gate.sh` checks that the probe's own log opens in
user://logs, native and in the sandbox, with a save and load before every
frame, and from states at frame 0 and frame 25 (the probe removes the folder
at frame 10, so the later state is seen to carry the disk as the game left
it, where a new machine has the folder); and that Dodge the Creeps, which
writes nothing, exports no save data. The probe was exported again with this
core's Godot editor for these checks. `tests/run-frontend.sh` makes a project
with a generated `stand-in.nes` in the files slot and plays it to the
reference's picture.

The game is a Godot 4.6 game and runs only in the Godot 4.6 core, where it
was played (that core's docs/PLAN.md); this core has the same two changes,
as every Godot core has the same files, and was tested with its own test
games.

Measured on jaffanator01 (2026-11-14), with the probe exported again by the
Godot 4.4.1 editor: `waterbox/run-gate.sh` 51 passed, 0 failed (the new
user://logs and save data checks among them); `tests/run-frontend.sh` 6 ok,
the new files slot check among them. The checks were seen to fail in the
Godot 4.6 core, without the folder and before the slot changed.
