# Third-party code under vendor/

Both files are vendored unmodified. Every adaptation TinyGB needs lives outside this directory: `core/tg_peanut.c` supplies the callbacks and the save-state serialiser, and the build sets `AUDIO_SAMPLE_RATE`, `MINIGB_APU_AUDIO_FORMAT_S16SYS`, `ENABLE_SOUND` and `ENABLE_LCD` from the makefiles. Warnings in these files are suppressed per object (`VENDOR_CFLAGS`) rather than fixed here.

Neither upstream file carries a version number. The identity of the vendored copies is their content hash; `make check-vendor` recomputes it.

| File | Licence | Copyright | Upstream | SHA-256 |
|---|---|---|---|---|
| `peanut_gb.h` | MIT | 2018-2023 Mahyar Koshkouei; parts from SameBoy, MIT, 2015-2019 Lior Halphon | https://github.com/deltabeard/Peanut-GB | `d4d64436c2e45075b3b60f44910fc9e7e824f2167e94503c775e6130e03f4196` |
| `minigb_apu.c` | MIT | 2019 Mahyar Koshkouei; 2017 Alex Baines (MiniGBS) | https://github.com/deltabeard/minigb_apu | `0e9e18cadc3fb0daf26fa985c3b3b49232bcf09d2909aeb1c4711c07b2800e53` |
| `minigb_apu.h` | MIT (the header refers to a LICENSE file that is not vendored; the `.c` states MIT in full) | as above | as above | `454707a08762191f8a3bd44d5b4e64f250cb43fc9cfb5b6fa0d5c4773a7ad1ab` |

The MIT text for both is the same as this repository's `LICENSE`, with the copyright lines above.

Upstream limits that TinyGB inherits (from the comments at the end of `peanut_gb.h`): MMM01 and MBC6 untested; MBC7, Pocket Camera, Bandai TAMA5, HuC3 and HuC1 unsupported; no Game Boy Color.
