# Xenoblade Chronicles 2 · 4K Scene, UI and Screen Effects Fixes

[简体中文](README.zh-CN.md)

Fork of [deemocrs/XB2_JP_3.0.0_1080p_UI_ScreenFX](https://github.com/deemocrs/XB2_JP_3.0.0_1080p_UI_ScreenFX) with the
scene resolution raised from 1920×1080 to 3840×2160, plus a variant for
[xc2-recomp](https://github.com/AksharDP/xc2-recomp). The ScreenFX fix addresses the translucent boxes in cutscenes
reported as [AksharDP/xc2-recomp#58](https://github.com/AksharDP/xc2-recomp/issues/58) (same family as
[#46](https://github.com/AksharDP/xc2-recomp/issues/46)).

The base-game patch targets version 3.0.0 of the Japanese/Chinese edition of Xenoblade Chronicles 2.

## Contents

| Folder | For | What it does |
| --- | --- | --- |
| `XB2_JP_3.0.0_1080p_UI_ScreenFX` | Ryujinx / yuzu / Atmosphere, JP/CN 3.0.0 | 1080p scene (1920×1080) + UIBounds + ScreenFX — the upstream patch |
| `XB2_JP_3.0.0_1440p_UI_ScreenFX` | Ryujinx / yuzu / Atmosphere, JP/CN 3.0.0 | 1440p scene (2560×1440) + UIBounds + ScreenFX |
| `XB2_JP_3.0.0_4K_UI_ScreenFX` | Ryujinx / yuzu / Atmosphere, JP/CN 3.0.0 | 4K scene (3840×2160) + UIBounds + ScreenFX |
| `xc2-recomp_UI_ScreenFX_fix` | [xc2-recomp](https://github.com/AksharDP/xc2-recomp) | UIBounds + ScreenFX only, one file each for JP 3.0.0 and western 3.0.1 |

Install only **one** resolution variant: all three patch the same two scene-init instructions.

### Patch contents (emulator mods)

| Component | What it changes |
| --- | --- |
| **Scene** | Two instructions at main+0x700640/0x700644, re-encoded per variant: 1920×1080 `15F08052`/`16878052`, 2560×1440 `15408152`/`16B48052`, 3840×2160 `15E08152`/`160E8152` (little-endian). |
| **UIBounds** | Unchanged from upstream: four instructions so UI bounds use the engine's logical width and height (fixes the selected Blade portrait position and border). Resolution-independent. |
| **ScreenFX** | Unchanged from upstream: four instructions that project the affected screen-space particles in their original 1280×720 coordinate system, so effects meant to cover the screen have no visible edges. The effects still draw at the render target's actual resolution. This is the fix for the cutscene boxes. |

### The xc2-recomp variant

xc2-recomp's built-in `xc2res` patch set rewrites the very same two scene-init instructions (main+0x700640/44) to read
the runtime resolution from its patch page — the resolution follows the `XC2_OUTPUT_RES` in-game setting. A mod patch
that touches those two instructions switches the whole built-in set off while the mod is on (see its
`docs/mods.md`), so this variant ships **without** the scene lines. Use the in-game resolution setting for 4K; the port
scales sizes down to fit the game's memory (a 4K output becomes about 3042×1712) — that part is the port's own
behaviour.

Both files in this variant carry only UIBounds + ScreenFX:

- `XC2_JP_3.0.0.pchtxt` — build id `FEEE7AFA8A1541BD67AED1A0061FBB57` (Japanese/Chinese 3.0.0)
- `XC2_3.0.1.pchtxt` — build id `4F5C96E2D97F45AC288BC67613EBEF0A` (Americas/Europe 3.0.1; its `.text` is byte-for-byte
  identical to JP 3.0.0 at the same offsets, per xc2-recomp's `tools/xc2_titles.py`)

## Installation

### Ryujinx / yuzu / Atmosphere (4K mod)

- Base game Title ID `0100F3400332C000`
- Build ID `FEEE7AFA8A1541BD67AED1A0061FBB57`

1. Stop the game. Download this repository with **Code → Download ZIP**.
2. In Ryujinx, right-click the game and open its mod directory. Copy **one** of the `XB2_JP_3.0.0_{1080p,1440p,4K}_UI_ScreenFX` subfolders into that directory (yuzu: load mod folder for the title; Atmosphere: `atmosphere/contents/...` on the SD card).
3. Enable the mod and restart the game. The log should show `Matching IPSwitch patch` with the patch filename.

```
<Game mod directory>/
└── XB2_JP_3.0.0_4K_UI_ScreenFX/          (or the 1080p / 1440p folder)
    └── exefs/
        └── XC2_JP_3.0.0_4K_UI_ScreenFX.pchtxt
```

### xc2-recomp

1. Open the `mods` folder next to `xc2.exe` (in game: F1 → Mods tab → **Open mods folder**).
2. Copy the whole `xc2-recomp_UI_ScreenFX_fix` folder into `mods\`. The game reads the file that matches your
   release and ignores the other.
3. Run the XC2-Setup installer on the install folder again — code patches are compiled into the game at setup time;
   until then the mod stays off.
4. Set the output resolution in game (F1) or `xc2.ini`.

```
mods/
└── xc2-recomp_UI_ScreenFX_fix/
    └── exefs/
        ├── XC2_JP_3.0.0.pchtxt
        └── XC2_3.0.1.pchtxt
```

## Notes and testing

- The scene re-encodings for all three variants were verified by ARM64 round-trip decode/encode; the UIBounds and
  ScreenFX bytes are untouched from the upstream 1080p mod, which its author validated offline at 720p, 1080p, 1440p
  and 4K (16:9), plus 1600×900 for ScreenFX.
- 4K in the *emulator* path means the render target initialization constant; whether the target GPU keeps 60 fps at
  3840×2160 is hardware-dependent. Ryujinx also needs its own resolution scaling settings to actually allocate the
  larger surface.
- On xc2-recomp, only UIBounds + ScreenFX are known to interact correctly with its built-in resolution system; the
  combined 4K scene mod is not meant for the port.

## Credits

- [deemocrs](https://github.com/deemocrs/XB2_JP_3.0.0_1080p_UI_ScreenFX) — the original 1080p patch, analysis and
  testing.
- [AksharDP/xc2-recomp](https://github.com/AksharDP/xc2-recomp) — the port, and issues
  [#58](https://github.com/AksharDP/xc2-recomp/issues/58) /
  [#46](https://github.com/AksharDP/xc2-recomp/issues/46) documenting the cutscene artifacts.
