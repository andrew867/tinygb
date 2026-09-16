# RetailOS Integration

How the RetailOS (NanoApps) build of TinyGB is produced from this repository and installed with NanoApps' own tooling, without changing that tooling.

## What NanoApps expects

`./start` and `tools/build_apps.py` discover an app as `apps/<name>/` containing a `Makefile` (so `start` treats it as a target), `Info.plist` (identity, icon), and after a build `build/<name>.hbapp`. `sdk/hb_app.mk` locates the SDK from its own path (`SDK_DIR := $(abspath $(dir $(lastword $(MAKEFILE_LIST))))`), so it can be included from any directory.

## The forwarder in the NanoApps fork

`NanoApps/apps/tinygb/` keeps three files and nothing else:

`Makefile`:

```make
# TinyGB lives in its own repository. This forwards NanoApps' build to it and
# copies the result to where build_apps.py looks. TINYGB points at the checkout.
TINYGB ?= ../../../tinygb
NANOAPPS := $(abspath ../..)

.PHONY: all clean
all:
	@$(MAKE) -s -C $(TINYGB)/retailos NANOAPPS=$(NANOAPPS)
	@mkdir -p build
	@cp -f $(TINYGB)/retailos/build/tinygb.hbapp build/tinygb.hbapp

clean:
	@$(MAKE) -s -C $(TINYGB)/retailos clean
	@rm -rf build
```

`Info.plist`: a copy of `retailos/Info.plist` (the packer reads it from the NanoApps side; keep the two identical, and the forwarder's `all` can `cp` it too).

`README.md`: two lines pointing at this repository.

With that, `./start build tinygb`, `./start install tinygb`, and `make -C apps tinygb` work as they do for every other app, and `./start install` (everything) includes TinyGB.

## `retailos/Makefile` in this repository

```make
APP_NAME    := tinygb
NANOAPPS   ?= ../../NanoApps
SRCS       := tinygb.c \
              ../core/tg_core.c ../core/tg_peanut.c ../vendor/minigb_apu.c \
              ../platform/tg_scale.c ../platform/tg_audio_clock.c ../platform/tg_tilt.c \
              ../platform/tg_palette.c ../platform/tg_pad.c ../platform/tg_text.c \
              ../platform/tg_settings.c ../platform/tg_build.c \
              ../platform/tg_audio_hb.c ../platform/tg_input_hb.c ../platform/tg_fs_hb.c \
              ../platform/tg_sys_hb.c ../ui/tg_menu_raw.c
RAW_SURFACE := 1
# Consumed by a := inside hb_app.mk, so it has to be set before the include.
EXTRA_CFLAGS := -O2 -DAUDIO_SAMPLE_RATE=22050 -DMINIGB_APU_AUDIO_FORMAT_S16SYS=1 \
                -I../platform/retailos/include \
                -Wno-sign-compare -Wno-implicit-fallthrough -Wno-unused-but-set-variable -Wno-type-limits \
                -DEN_BUILD_STAMP='"$(shell date -u +%Y%m%d.%H%M)"' -DEN_BUILD_GIT='"$(shell git rev-parse --short=7 HEAD)"'
include $(NANOAPPS)/sdk/hb_app.mk
```

Plus a post-link size check (`stat -c%s build/tinygb.hbapp`, fail over 524288) appended after the include as an extra prerequisite of `all`. Whether `-mcpu=cortex-a5 -mfpu=vfpv4` is added is OQ-008.

Note the single `gcc` invocation in `hb_app.mk`: there are no per-object rules, so the vendor warning suppressions apply to everything; that is accepted on this target.

## Data on the device

`/Apps/Data/TinyGB/roms/` holds cartridges. NanoApps' `./start install --demo-data` copies `data/<Name>/` from the NanoApps tree; TinyGB ships no cartridges, so the folder is created by the app on first launch and populated by the owner in disk mode. `NanoApps/data/TinyGB/roms/README.txt` (one line: put your cartridges here) can be added to the fork so the demo-data copy makes the folder visible.

## Debugging

- `./start trace` prints the DRAM trace ring: TinyGB logs `INIT`, heap headroom, cartridge start, sink start/stop, and every refused file.
- Home triple-click takes a screenshot through the resident; `./start pull media` fetches it. This is how AC-ROS-001 is checked.
- A device that hangs on launch is almost always a `.hbapp` over the ceiling or an arena that did not fit: check the printed size and the last trace entries.

## Verification of the integration

`./start build tinygb` from the fork produces `apps/tinygb/build/tinygb.hbapp`; `./start install tinygb` puts the icon on the Home Screen with the gamepad glyph and green colour; MAN-SMOKE-001.
