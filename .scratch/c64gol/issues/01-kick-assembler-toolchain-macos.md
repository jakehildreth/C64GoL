# Research: Kick Assembler toolchain on macOS (arm64)

Type: research
Status: claimed
Blocked by:

## Question

What is the exact, working way to install and run Kick Assembler on macOS arm64 (this machine), sufficient to assemble a `.prg` from the command line?

Resolve:
- Current Kick Assembler version and where to get it (official site, brew cask, other).
- Its Java dependency: which JRE/JDK works on macOS arm64, and whether any are already installed on this machine (`/usr/libexec/java_home -V`).
- The exact command line to assemble `main.asm` into a `.prg` (including any `-o`, `-log`, symbol-file flags we'd want in a build script).
- Whether any macOS-specific quirks exist (Gatekeeper/quarantine on the jar, path issues).
- Recommend the build invocation to bake into a `build.sh` (or Makefile) in this repo.

Record findings as the answer on this ticket. Do not install anything system-wide without noting it in the answer.
