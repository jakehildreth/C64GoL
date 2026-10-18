# Research: Kick Assembler toolchain on macOS (arm64)

Type: research
Status: resolved
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

## Answer

**Verdict: works out of the box on this machine.** Kick Assembler 5.25 was downloaded, run against the Homebrew OpenJDK 21 already installed here, and assembled a test `main.asm` into a valid CBM `.prg` (verified byte-for-byte, `$0801` BASIC SYS upstart header) with zero workarounds. Nothing was installed system-wide during this research.

### 1. Current version & where to get it

- **Version: V5.25** (released 18 Nov 2022; latest as of 2026-10-17). Author: Mads Nielsen.
- **Only distribution channel: the official site** — https://theweb.dk/KickAssembler/ → direct zip: https://theweb.dk/KickAssembler/KickAssembler.zip (~1.3 MB).
- **No Homebrew cask/formula exists** (verified: `brew search kick` and `brew info --cask kick-assembler` → "No Cask with this name exists").
- Zip contents (verified by download): `KickAss.jar` (~640 KB, the whole assembler), `KickAss.cfg` (default options: `-showmem`, `-symbolfile`), `KickAssembler.pdf` (manual), `Examples/`.
- Install = unzip anywhere user-writable, e.g. `tools/kickassembler/`. No installer, no root needed.

### 2. Java dependency & what's on this machine

- **Requirement: Java 8 or higher** (manual, ch. 2.1: "Kick Assembler run on any platform with Java8.0 or higher"). The site links AdoptOpenJDK 11, but any modern JDK works — empirically confirmed here with OpenJDK 21.
- **This machine's state (verified 2026-10-17):**
  - `/usr/libexec/java_home -V` → **fails**: "Unable to locate a Java Runtime". No JDK is registered with macOS's system `java_home`, so a bare `java` command does NOT work.
  - **BUT Homebrew has two working JDKs already installed:**
    - `/opt/homebrew/opt/openjdk/bin/java` → OpenJDK **27** (arm64, Homebrew build)
    - `/opt/homebrew/opt/openjdk@21/bin/java` → OpenJDK **21.0.12.1** (arm64, LTS)
  - Neither is linked into `/Library/Java/JavaVirtualMachines` (that step needs `sudo`; NOT done, not needed).
- **Recommendation: use the LTS one, `$(brew --prefix openjdk@21)/bin/java`.** It runs the jar natively on arm64 (no Rosetta). Do NOT brew-install anything else; do NOT do brew's `sudo ln` system-JVM step.

### 3. Exact command line

Verified working on this machine (test `.asm` with `BasicUpstart2` → produced `main.prg` 23 bytes, `file` reports "CBM BASIC, SYS 2062"):

```
"$(brew --prefix openjdk@21)/bin/java" -jar tools/kickassembler/KickAss.jar main.asm -o main.prg -log build.log -symbolfile -vicesymbols
```

Flags worth baking in (from the official Quick Reference, http://theweb.dk/KickAssembler/webhelp/content/cpt_QuickReference.html):
- `-o <file>` — output file; default is input name with `.prg` suffix. Default `.prg` output already includes the 2-byte load address (that's what VICE/`x64sc` and `LOAD`/`RUN` want; `-binfile` would strip it — don't use).
- `-log <file>` — assembler output to a logfile.
- `-symbolfile` — writes `main.sym` (KickAsm-format symbols, re-importable via `.import source`).
- `-vicesymbols` — writes `main.vs` (VICE monitor label file — we'll want this for debugging in VICE).
- `-showmem` — memory map after assembly (already in the zip's default `KickAss.cfg`; note the cfg only takes effect if it sits **next to the jar**).
- Also available if needed later: `-odir <dir>`, `-define DEBUG`, `-execute "x64sc ..."`, `-debugdump` (C64Debugger mapping), `-libdir`.

### 4. macOS quirks

- **Gatekeeper/quarantine: NOT a problem.** `java -jar` is not subject to Gatekeeper app-launch checks. Tested end-to-end on this machine: the quarantined zip was downloaded with `curl`, unpacked with `unzip`, and the jar ran immediately — no "damaged app" dialog, no `xattr -d com.apple.quarantine` needed. (Note: unpacking with Finder instead of `unzip` could attach quarantine to the jar; if it ever trips, `xattr -d com.apple.quarantine KickAss.jar` clears it, or just always unzip from the shell.)
- **Bare `java` is broken here** (system `java_home` empty) — the single real path gotcha. Fix in scripts: resolve `"$(brew --prefix openjdk@21)/bin/java"` rather than relying on `PATH`. Homebrew openjdk is "keg-only" and deliberately not symlinked into `/opt/homebrew/bin`.
- No Rosetta needed; jar is pure bytecode and runs on the arm64 JVM.
- No spaces-in-path issues as long as the java path and jar path are quoted.

### 5. Recommended `build.sh` invocation

```bash
#!/usr/bin/env bash
set -euo pipefail
cd "$(dirname "$0")"

JAVA="$(brew --prefix openjdk@21)/bin/java"   # already installed; keg-only LTS JDK
KICKASS="tools/kickassembler/KickAss.jar"     # unzip KickAssembler.zip here

"$JAVA" -jar "$KICKASS" main.asm \
  -o main.prg \
  -log build.log \
  -symbolfile \
  -vicesymbols \
  -showmem
```

Produces `main.prg` (run in VICE via `x64sc main.prg`), `main.sym`, `main.vs`, `build.log`. Keep `KickAss.jar` out of git (add `tools/kickassembler/` to `.gitignore`) or vendor the zip; jar is freeware redistributable with credit.

### Sources

- Official site & download: https://theweb.dk/KickAssembler/
- Manual, Getting Started (Java 8+, `java -jar kickass.jar`): http://theweb.dk/KickAssembler/webhelp/content/cpt_GettingStarted.html
- Manual, Quick Reference (all CLI flags): http://theweb.dk/KickAssembler/webhelp/content/cpt_QuickReference.html
- Version date corroboration: https://www.c64-wiki.com/wiki/KickAssembler
- Local verification: zip downloaded to `/tmp`, assembled test file with `/opt/homebrew/opt/openjdk@21/bin/java` (exit 0, valid .prg bytes); `/usr/libexec/java_home -V`, `java -version`, `brew list`, `brew search kick` outputs as quoted above.
