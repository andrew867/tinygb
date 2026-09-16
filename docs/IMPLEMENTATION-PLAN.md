# Implementation Plan

## Assumptions

- The repository is `andrew867/tinygb` on GitHub, public, MIT, with the vendored MIT notices kept. Local checkout `C:\src\tinygb`, sibling of `C:\src\NanoApps` and `C:\src\ipod`.
- The history of `apps/tinygb` in the NanoApps fork (18 commits, 2026-09-03 to 09-08) has already been extracted with `git filter-repo` into that checkout (Phase 0 below is done as of 2026-09-16, locally, unpushed).
- Development happens on Windows with WSL Debian 13: gcc 14 for host tests, `arm-none-eabi-gcc` 14.2 for RetailOS, the musl cross toolchain in `C:\src\ipod\artifacts\linux-n31` for N31. All present.
- One iPod nano 7G is available for device tests and can be switched between RetailOS (NanoApps installed) and N31.
- Peanut-GB stays the only core through this plan. Colour is deferred.
- The N31 build keeps working at every phase; it is the regression baseline for the portable code.

## Repository structure (target)

```
tinygb/
  README.md  LICENSE  CHANGELOG.md  Makefile  .gitignore
  .github/workflows/ci.yml
  core/          tg_core.h  tg_core.c  tg_peanut.c
  vendor/        peanut_gb.h  minigb_apu.c  minigb_apu.h  LICENSES.md
  platform/      tg_scale.*  tg_audio_clock.*  tg_tilt.*  tg_palette.*  tg_pad.*  tg_text.*  tg_settings.*
                 tg_save.h  tg_roms.h            (interfaces)
                 tg_save.c  tg_roms.c            (POSIX: host + n31)
                 tg_fb.h  tg_fb_linux.c  tg_audio.h  tg_audio_alsalib.c  tg_audio_linux.c  tg_input.h  tg_input_linux.c
                 tg_audio_hb.c  tg_input_hb.c  tg_fs_hb.c  tg_sys_hb.c   (RetailOS)
                 retailos/include/time.h        (struct tm shim for the vendored core)
  ui/            tg_menu.h  tg_menu.c (LVGL, n31)  tg_menu_raw.c (RetailOS)
  linux/         tg_linux.c  Makefile.n31  lv_conf_n31.h
  linux/shim/    drmfb.c/h  fbcon.c/h  touch.c/h  fbrefresh.h  build_stamp.c/h  lvgl-mirror.mk  PROVENANCE
  retailos/      tinygb.c  Info.plist  Makefile
  host/          tg_headless.c  tests.c  Makefile.host
  tools/         fetch-testroms.sh  gate.sh  acid2check.py  grab-screen.sh  sync-n31-shims.sh
  docs/          this set
  testroms/      fetched, ignored
```

Kept as-is from the extracted tree: `core/`, `vendor/`, `platform/` (flat, suffix convention `_linux` / `_alsalib` / `_hb`), `ui/`, `linux/`, `host/`, `tools/`. New: `retailos/`, `linux/shim/`, `docs/`, the top-level `Makefile`, CI. The existing root `Makefile`, `Info.plist`, and `tinygb.c` move into `retailos/` (they are the RetailOS app's files and nothing else).

## Phase 0: Extraction (done locally 2026-09-16)

**Goal**: a standalone git repository holding only TinyGB's history.

Done: `git clone --no-local NanoApps tinygb; git filter-repo --path apps/tinygb --path-rename apps/tinygb/:`. Result: 18 commits, 43 files, 170 KB pack, no remote.

Remaining tasks (need the go-ahead, because they are outward-facing):
- Create `andrew867/tinygb` on GitHub (public), push `main`.
- Nothing is removed from the NanoApps fork until Phase 2 proves the N31 build from the new repo.

**Completion**: the GitHub repo exists with the extracted history and this `docs/` set.

## Phase 1: Repo skeleton, host build, CI

**Goal**: `make test` and `make gate` pass from a fresh clone on Linux, and CI runs them.

Files: `Makefile` (new; targets `host`, `test`, `gate`, `n31`, `retailos`, `clean`, dispatching to `host/Makefile.host`, `linux/Makefile.n31`, `retailos/Makefile`), `.github/workflows/ci.yml`, `README.md`, `LICENSE`, `vendor/LICENSES.md`, `.gitignore` (add `out/`, `build*/`, `testroms/`, `*.sav`, `*.st0`).

Tasks:
1. Move `Makefile`, `Info.plist`, `tinygb.c` to `retailos/` (git mv).
2. Add `tg_palette.c` to the host test link and a test for `tg_palette_index` and the clamp (it was never linked).
3. Add the `nm`-based check that no `gb_*` symbol is referenced outside `tg_peanut.o` (AC-CORE-004) as a `make check-seam` target.
4. CI: fetch test ROMs (they are downloads, not committed), build host, run tests and gate, upload `dmg-acid2.ppm` as an artifact. Cache nothing.
5. Write `vendor/LICENSES.md` naming Peanut-GB and minigb_apu, their copyright lines, and that no local changes exist.

**Completion**: CI green on the first push; `make gate` prints `pass 3 fail 0 skip 0`.

## Phase 2: N31 parity from the new repo

**Goal**: the N31 binary built from `C:\src\tinygb` is functionally identical to the one built from NanoApps, and the ipod tree builds it from the new location.

Files: `linux/Makefile.n31` (paths), `linux/shim/*` (new, vendored), `tools/sync-n31-shims.sh` (new), `linux/tg_linux.c` (`:913` return-code fix), `host/lv_conf_n31.h` and `platform/tg_scale.h` (Cortex-A8 comments), `C:\src\ipod\tools\linux-n31\build-n31-apps.sh`, `install-n31os-disk.ps1` (paths), `docs/N31-INTEGRATION.md`.

Tasks:
1. Copy the five launcher files, `build_stamp.c/h`, and `lvgl-mirror.mk` into `linux/shim/` with a `PROVENANCE` file (source path, NanoApps commit `08eed61`). `sync-n31-shims.sh` re-copies from `$NANOAPPS` and prints a diff; it is run by hand, not by the build.
2. Rewrite the include and source paths in `Makefile.n31` to `../linux/shim` and repo-relative `../platform`; keep every flag, the `check-built` assertions, the stamp rule, and the staging copy exactly.
3. Build, run `check-built`, push to the device, play Tetris and one battery game; capture `/dev/fb0` for dmg-acid2 and compare to `tg_headless -S` (AC-N31-002).
4. Re-point `build-n31-apps.sh` (`build_tinygb` -> `make -C "$TINYGB/linux" -f Makefile.n31`, with `TINYGB=${TINYGB:-$(dirname "$ROOT")/tinygb}`) and `install-n31os-disk.ps1` (binary path). `mk-release-zip.sh` reads from artifacts and needs no change.
5. Fix REQ-N31-060 and REQ-N31-061.

**Completion**: `bash tools/linux-n31/build-n31-apps.sh tinygb` in the ipod tree builds from the new repo; AC-N31-001..004 pass; the NanoApps fork's `apps/tinygb` can now be reduced to the forwarder (Phase 3, step 1).

## Phase 3: RetailOS picture and input

**Goal**: a cartridge runs on RetailOS with the right picture and playable controls, silent.

Files: `retailos/tinygb.c` (rewrite), `retailos/Makefile`, `retailos/Info.plist`, `platform/tg_input_hb.c`, `platform/tg_fs_hb.c`, `platform/tg_sys_hb.c`, `platform/retailos/include/time.h`, `platform/tg_pad.c` (decouple from `tg_fb`: take pixels + stride), `platform/tg_tilt.c` (drop `snprintf`), `platform/tg_text.c` (new), NanoApps fork `apps/tinygb/{Makefile,Info.plist,README.md}` (forwarder).

Tasks:
1. Forwarder in NanoApps (`RETAILOS-INTEGRATION.md`), so `./start build tinygb` invokes `make -C $TINYGB/retailos NANOAPPS=...` and copies the `.hbapp` to `apps/tinygb/build/`.
2. Freestanding pass: compile `core/` + portable `platform/` with the SDK flags; fix `snprintf`, `time.h`, `abort`; confirm `memmove` is not emitted or include the shim.
3. `hb_raw_init` paints, makes `/Apps/Data/TinyGB/roms`, scans, shows the list with `tg_text`; a tapped row reads the ROM into the 1 MiB window and starts it (audio rate 0 for now).
4. `hb_raw_frame`: one frame per tick, scale into `hb_raw_fb()` at y = 0, pad drawn below, inputs from `tg_input_hb`.
5. Log `hb_os_heap_largest/free` at init to the trace ring; read them back with `./start trace`; decide OQ-004 and OQ-008 from measurements (frame time at `-Os` vs `-O2`, `neon` vs `vfpv4`, jump tables). Record in `OPEN-QUESTIONS.md`.
6. Screenshot dmg-acid2 via triple-click, pull, compare (AC-ROS-001).

**Completion**: AC-ROS-001, AC-ROS-003, AC-ROS-006 pass; Tetris is playable silently at full speed for five minutes with no visible stutter.

## Phase 4: RetailOS audio

**Goal**: sound, paced correctly, with pause/resume that does not click.

Files: `platform/tg_audio_hb.c` (new), `retailos/tinygb.c` (pacing decision), `host/tests.c` (slot state machine + duration exactness tests with fakes).

Tasks:
1. Implement the slot queue per `SPEC-retailos-audio.md` with a build-time test-tone generator switch.
2. Tone first: 60 s of 440 Hz recorded through headphones, checked for gaps (AC-AUD-001).
3. Switch to `core->audio_pull`; implement the 0/1/2-frames-per-tick decision from queue depth; tune `TG_AUD_SLOT_FRAMES`, `TG_AUD_SLOTS`, `TG_AUD_TARGET_LEAD` on the device; record the numbers (OQ-010).
4. Pause/resume/stop with the in-place silence; menu pill wiring can be a placeholder tap zone for now.
5. Media re-pause poll; starvation restart.

**Completion**: AC-AUD-001..004 and AC-ROS-002 pass.

## Phase 5: RetailOS menu, saves, settings, polish

**Goal**: the app is complete for a player.

Files: `ui/tg_menu_raw.c` (new), `platform/tg_settings.c` (new, shared with N31 later), `platform/tg_fs_hb.c` (saves, states), `retailos/tinygb.c` (modes), `linux/tg_linux.c` (adopt `tg_settings` so both targets read the same file format).

Tasks:
1. Menu pages and rows per REQ-ROS-031/032; About row with `en_build_version()` (the build stamp mechanism moves into a portable `platform/tg_build.c` and both Makefiles set the defines).
2. Battery save cadence 2 s; flush on menu; state on pause; Resume in the library.
3. Settings file; palette/smooth/tilt live changes with full repaint.
4. Wake-lock poke; empty-library help screen; the "too big" listing state.
5. Overlay (fps, queue depth, underruns) behind a Settings toggle.

**Completion**: AC-ROS-004, -005, -007, -008 and AC-DATA-001..005 pass; MAN-ROS-* all Pass.

## Phase 6: Hardening and release

**Goal**: something a stranger can install.

Tasks:
1. Size audit: print and gate the `.hbapp` size; strip the overlay font to the glyphs used if needed.
2. Ten-launch leak test (AC-ROS-007); one-hour soak on each target.
3. Update `README.md` with screenshots pulled from the device; `CHANGELOG.md` 0.1.0.
4. Tag `v0.1.0`; N31 release zip picks up the new binary (no change needed); NanoApps fork README lists TinyGB with a link to this repo.
5. Remove the last copies of the emulator source from the NanoApps fork, leaving the forwarder.

**Completion**: `QA-CHECKLIST.md` fully ticked.

## Recommended build order and dependencies

0 -> 1 -> 2 -> 3 -> 4 -> 5 -> 6. Phase 2 must precede 3 only because it is the cheapest proof that the portable code survived the move; 3 and 4 are strictly sequential (audio pacing needs a working picture and input to test). The forwarder (3.1) can be done during 2.

## Technical decisions

| Decision | Choice | Why |
|---|---|---|
| Layout | keep the flat `platform/` with `_linux` / `_hb` suffixes; add `retailos/` and `linux/shim/` | least churn; the suffix convention already exists; git keeps history through renames |
| Launcher shims | vendor with provenance + sync script | the new repo must build without NanoApps; the files are the user's own and change rarely |
| NanoApps coupling | one forwarder dir in the fork | keeps `./start`, `build_apps.py`, and `mkrelocapp.py` untouched |
| ROM storage on RetailOS | static 1 MiB `.bss` window | free on disk, no leak, no allocator that can fail after launch; measured before fixed |
| Audio sink | own slot queue, not Entrain's module | opposite clock master, tenth of the latency, no 1 s fade |
| Text on RetailOS | own 6 x 8 bitmap font | raw surface has no text; ~760 bytes |
| Menu on RetailOS | hand-drawn against `tg_menu.h` | no LVGL on the raw path; same interface keeps the N31 menu untouched |
| Optimisation flags | `-O2` via `EXTRA_CFLAGS`, FPU flag by measurement | `-Os` on an in-order core is measurably slower for an emulator; NEON is absent on the part |

## What to stub first

Audio: a tone generator behind a build switch, before the APU. Menu: a tap zone that toggles pause, before the pages. Saves: the whole-file write path, before the cadence.

## What not to build yet

Colour (second core), Bluetooth pads, link cable, multiple state slots, RTC, screenshots from inside the app, landscape.

## Deferred Work

- **Phase 07 (colour)**: a SameBoy-backed `tg_core` table. The registry, the pixel format (palette index), and `tg_core_for_rom` already anticipate it. Memory on RetailOS (SameBoy's context is much larger) is the first thing to size.
- **Phase 06 (Bluetooth pad on N31)**: a new `TG_SRC_*` bit in `tg_input_linux.c`; nothing else changes.
- **Frameskip**: a core hint to skip PPU rendering when the RetailOS tick cannot keep up. Peanut-GB has `frame_skip`; expose it if Phase 3 measurements demand.
- **Shared settings module on N31**: `tg_settings` replaces the menu-only state; small.
