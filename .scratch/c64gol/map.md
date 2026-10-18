# Map: C64 Game of Life

## Destination

A PRG of Conway's Game of Life for the Commodore 64 that runs correctly when dropped into c64online.com: boots to a pattern-library menu with live previews, every canonical pattern behaves correctly (glider glides, blinker blinks, gun fires, auto-stop triggers), and speed/pause/quit controls work. The map is done when nothing is left to decide before building.

## Notes

- Domain: Commodore 64 assembly (6502), Kick Assembler toolchain, character-mode graphics.
- Skills every session should consult: `grilling` + `domain-modeling` for decision tickets; `research` for research tickets; `prototype` for spike tickets.
- Tracker: local markdown under `.scratch/c64gol/` (see `docs/agents/issue-tracker-local.md` conventions from the wayfinder skill).
- No PR is opened for `research/` branches; they are never merged.
- No emojis in code or docs. CalVer for any versioning. Commit messages: conventional commits, drafted for Jake's approval before committing.

## Decisions so far

_(grilled and settled during charting, 2026-10-17)_

- **Language/toolchain**: 6502 assembly via Kick Assembler.
- **Output format**: PRG; must load by drag-and-drop on c64online.com (user-confirmed accepted).
- **Display**: quarter-block (quadrant) glyphs from the stock uppercase/graphics charset; 80×46 logical canvas (2×2 cells per character over 40×23 character cells) + 2-row menu at character rows 23–24. Verified the charset carries the quadrant-block family (screen codes ~$6B–$7F plus reverse mirrors). No custom charset.
- **Interaction**: preset library first (no live-editing). Menu screen (list + metadata + live quadrant preview) → run on canvas → keypress returns to menu.
- **Pattern library**: the canon (~9: glider, blinker, toad, beacon, pulsar, lightweight spaceship, Gosper glider gun, R-pentomino, acorn), stored as sparse cell-coordinate lists; all fit 80×46.
- **Run end**: auto-stop on all-dead or exact static repeat; keypress always returns to menu; no period-2+ detection.
- **Speed**: runtime adjustable (+/- keys) plus pause; top setting unthrottled.
- **Grid size**: locked at 80×46, not configurable.
- **Grid wrap**: build-time constant — hard walls (default) or torus.
- **Cell encoding**: build-time constant — byte-per-cell double buffer (default) or bit-packed.
- **Canvas menu (2 rows)**: status-rich — pattern name, generation counter, live-cell count, current speed, one legend line (`SPC pause  +/- speed  Q quit`).
- [Research: Kick Assembler toolchain on macOS (arm64)](.scratch/c64gol/issues/01-kick-assembler-toolchain-macos.md): KickAss 5.25 zip from theweb.dk (no brew cask) runs fine on already-installed Homebrew openjdk@21; build.sh calls `"$(brew --prefix openjdk@21)/bin/java" -jar KickAss.jar main.asm -o main.prg -log build.log -symbolfile -vicesymbols`.

## Not yet specified

- **Build-time variant validation**: whether torus-wrap and bit-packed builds are ever assembled/smoke-tested, and whether they ship in the same PRG build matrix or stay as latent options. Only the default build (hard walls, byte-per-cell) is required by the destination.
- **Exact quadrant glyph mapping**: the precise 16 screen codes for the quadrant patterns (which code = which 2×2 bit pattern) — a lookup-table detail to pin at build time from a screen-code chart.
- **Kick Assembler acquisition**: exact version/source on macOS (brew? manual jar? Java dependency) — a task-level detail for the toolchain ticket's resolution.
- **PRG load address / BASIC stub**: standard $080d-with-SYS-line vs bare $080d; trivial, decided at build time.
- **Key input method**: KERNAL GETIN ($FFE4) polling vs keyboard matrix scan — affects responsiveness of pause/speed/quit; a build-time decision within the input ticket's scope.

## Out of scope

- **Live cell editing on the canvas** (draw-your-own patterns): user chose preset-library-first; editing may return as a fresh effort after this one.
- **Bitmap-mode rendering** (320×200 pixel-per-cell): ruled out during display decision in favor of quadrant glyphs.
- **Half-block rendering** (40×N canvas): considered and rejected in favor of quarter-blocks.
- **Period-2+ oscillation detection**: auto-stop covers all-dead and exact static only.
- **Real-hardware / VICE verification**: destination is c64online.com only.
- **40×40 grid / alternate grid sizes**: grid locked at 80×46.
- **Runtime-configurable wrap/encoding**: build-time only.
