# Open Questions

| ID | Question | Area | Impact | Needed by | Status |
|---|---|---|---|---|---|
| OQ-001 | Repository name and visibility: `andrew867/tinygb`, public, MIT? (Matches `tinypod`.) | Repo | Low | Milestone 1 | yes MIT |
| OQ-002 | Vendor the launcher shims (`drmfb`, `fbcon`, `touch`, `build_stamp`, `lvgl-mirror.mk`) with a sync script, or keep reading them from `$NANOAPPS` at build time? | N31 build | Medium | Phase 2 | vendor; the repo must build alone but NanoApps #1 priority |
| OQ-003 | Where does the ipod tree expect the standalone repo: `$(dirname ROOT)/tinygb` like `NanoApps`, overridable with `TINYGB=`? | N31 integration | Low | Phase 2 | yes |
| OQ-004 | ROM storage on RetailOS: static 1 MiB `.bss` window (simple, no leak, arena risk) vs `hb_os_alloc` with a fixed DRAM mailbox recording the block (no arena risk, needs an unclaimed address). Decide after logging `hb_os_heap_largest()` on the device. | RetailOS memory | High | Phase 3 | window; measure first |
| OQ-005 | Map Play/Pause to Start with the `hb_media_state` re-pause trick, or leave it unmapped? | RetailOS input | Low | Phase 5 |  unmapped - touch on screen controls #1 priority |
| OQ-006 | Which colour core: SameBoy (accurate, large context) or another? Not needed for v0.1. | Core | Low now | Deferred | Deferred |
| OQ-007 | Pin the vendored Peanut-GB and minigb_apu by upstream commit hash in `vendor/LICENSES.md`, since neither file carries a version? | Vendor | Low | Phase 1 | Open (default: record the SHA-256 of each file and the upstream URL; upstream commit unknown) |
| OQ-008 | RetailOS compiler flags: keep the SDK's `-mcpu=cortex-a8 -mfpu=neon` or override with `-mcpu=cortex-a5 -mfpu=vfpv4` (the part has no NEON per N31); `-O2` vs `-Os`; `-fjump-tables` for the opcode switch? Decide by measured frame time and a working device. | RetailOS build | Medium | Phase 3 | `-mcpu=cortex-a5 -mfpu=vfpv4` |
| OQ-009 | Ship the fps / queue-depth overlay behind a Settings toggle in the release, or debug builds only? | RetailOS UI | Low | Phase 5 | Settings toggle, off)|
| OQ-010 | Final audio constants: `TG_AUD_SLOT_FRAMES`, `TG_AUD_SLOTS`, `TG_AUD_TARGET_LEAD`. Starting at 1470 / 6 / 2. | RetailOS audio | Medium | Phase 4 | Open |
| OQ-011 | Use the voice completion callback (proved by harness T3b) instead of polling `+0x64`, to reclaim within the tick it finishes? | RetailOS audio | Low | Phase 4 | Open (default: poll first) |
| OQ-012 | Should RetailOS auto-load the pause state on relaunch, or always ask? | Persistence | Low | Phase 5 |  ask via a Resume row |
| OQ-013 | Keep `tinygb` in the NanoApps fork README's app list with a link, or list it only in this repo? | Docs | Low | Phase 6 | list with a link |
| OQ-014 | Does upstream `nfzerox/NanoApps` want TinyGB submitted as an app (it would need the forwarder to become a real app dir with vendored sources), or does it stay fork-only? | Community | Low | After v0.1 | submitted as app so users can pull and build |
