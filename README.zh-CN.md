# 异度之刃2 · 4K 场景、UI 与屏幕特效修复

[English](README.md)

[deemocrs/XB2_JP_3.0.0_1080p_UI_ScreenFX](https://github.com/deemocrs/XB2_JP_3.0.0_1080p_UI_ScreenFX) 的 fork：场景分辨率从 1920×1080 提高到 3840×2160，并为
[xc2-recomp](https://github.com/AksharDP/xc2-recomp) 提供专用变体。ScreenFX 修复对应
[AksharDP/xc2-recomp#58](https://github.com/AksharDP/xc2-recomp/issues/58) 报告的过场半透明方块问题（同族问题见
[#46](https://github.com/AksharDP/xc2-recomp/issues/46)）。

本体补丁面向异度之刃2 日文/中文版 3.0.0。

## 内容

| 文件夹 | 适用 | 作用 |
| --- | --- | --- |
| `XB2_JP_3.0.0_1080p_UI_ScreenFX` | Ryujinx / yuzu / Atmosphere，日/中版 3.0.0 | 1080p 场景（1920×1080）+ UIBounds + ScreenFX — 即上游补丁 |
| `XB2_JP_3.0.0_1440p_UI_ScreenFX` | Ryujinx / yuzu / Atmosphere，日/中版 3.0.0 | 1440p 场景（2560×1440）+ UIBounds + ScreenFX |
| `XB2_JP_3.0.0_4K_UI_ScreenFX` | Ryujinx / yuzu / Atmosphere，日/中版 3.0.0 | 4K 场景（3840×2160）+ UIBounds + ScreenFX |
| `xc2-recomp_UI_ScreenFX_fix` | [xc2-recomp](https://github.com/AksharDP/xc2-recomp) | 仅 UIBounds + ScreenFX，日版 3.0.0 与美/欧 3.0.1 各一个文件 |

三个分辨率变体**只能装一个**：它们改的是同两条场景初始化指令。

### 模拟器补丁内容

| 组件 | 改动 |
| --- | --- |
| **Scene** | main+0x700640/0x700644 两条指令，按变体重编码：1920×1080 `15F08052`/`16878052`、2560×1440 `15408152`/`16B48052`、3840×2160 `15E08152`/`160E8152`（小端）。 |
| **UIBounds** | 与上游一致：四条指令让 UI 边界使用引擎逻辑宽高（修复战斗中选中等异刃头像及其边框的位置偏移）。与分辨率无关。 |
| **ScreenFX** | 与上游一致：四条指令让受影响的屏幕空间粒子按其原始 1280×720 坐标系投影，本应铺满屏幕的特效不再露出边缘；特效本身仍按渲染目标实际分辨率绘制。**这就是过场方块的修复。** |

### xc2-recomp 变体

xc2-recomp 内置的 `xc2res` 补丁组改写的正是同两条场景初始化指令（main+0x700640/44），改为从其补丁页读取运行时分辨率——分辨率跟随游戏内 `XC2_OUTPUT_RES` 设置。而 mod 一旦触碰这两条指令，内置补丁组在 mod 开启期间会整体关闭（见其
`docs/mods.md`），所以本变体**不带**场景两条指令。4K 请用游戏内分辨率设置；该移植版会把尺寸缩到游戏内存上限内（4K 输出实际约 3042×1712）——这是移植版自身的行为。

本变体只含 UIBounds + ScreenFX：

- `XC2_JP_3.0.0.pchtxt` — build id `FEEE7AFA8A1541BD67AED1A0061FBB57`（日/中版 3.0.0）
- `XC2_3.0.1.pchtxt` — build id `4F5C96E2D97F45AC288BC67613EBEF0A`（美/欧 3.0.1；其 `.text` 与日版 3.0.0 同偏移逐字节相同，依据 xc2-recomp 的 `tools/xc2_titles.py`）

## 安装

### Ryujinx / yuzu / Atmosphere（4K 补丁）

- 本体 Title ID `0100F3400332C000`
- Build ID `FEEE7AFA8A1541BD67AED1A0061FBB57`

1. 关闭游戏。用 **Code → Download ZIP** 下载本仓库。
2. Ryujinx：右键游戏打开 mod 目录；yuzu：打开该游戏的 Mod 数据目录；Atmosphere：复制到 SD 卡 `atmosphere/contents/...`。把 `XB2_JP_3.0.0_{1080p,1440p,4K}_UI_ScreenFX` 中**选一个**子文件夹复制进去。
3. 启用 mod 并重启游戏。日志应显示 `Matching IPSwitch patch` 及补丁文件名。

```
<游戏 mod 目录>/
└── XB2_JP_3.0.0_4K_UI_ScreenFX/          （或 1080p / 1440p 目录）
    └── exefs/
        └── XC2_JP_3.0.0_4K_UI_ScreenFX.pchtxt
```

### xc2-recomp

1. 打开 `xc2.exe` 旁边的 `mods` 文件夹（游戏内：F1 → Mods 标签 → **Open mods folder**）。
2. 把整个 `xc2-recomp_UI_ScreenFX_fix` 文件夹复制进 `mods\`。游戏会读取与你版本匹配的文件，忽略另一个。
3. 在安装目录重新运行 XC2-Setup 安装器——代码补丁在安装时编译进游戏；在此之前整个 mod 保持关闭。
4. 分辨率在游戏内（F1）或 `xc2.ini` 设置。

```
mods/
└── xc2-recomp_UI_ScreenFX_fix/
    └── exefs/
        ├── XC2_JP_3.0.0.pchtxt
        └── XC2_3.0.1.pchtxt
```

## 说明与测试

- 三个变体的场景指令字均经过 ARM64 解码/编码双向往返验证；UIBounds 与 ScreenFX 字节与上游 1080p 补丁完全一致，其作者已在
  720p/1080p/1440p/4K（16:9）及 ScreenFX 的 1600×900 下离线验证。
- 模拟器路径的「4K」指渲染目标初始化常数；3840×2160 下能否保住 60 fps 取决于硬件。Ryujinx 还需配合自身的分辨率缩放设置才会真正分配更大的画布。
- xc2-recomp 上，已知与内置分辨率系统正确互动的只有 UIBounds + ScreenFX；4K 场景完整版 mod 不是给移植版用的。

## 致谢

- [deemocrs](https://github.com/deemocrs/XB2_JP_3.0.0_1080p_UI_ScreenFX) — 原始 1080p 补丁、分析与测试。
- [AksharDP/xc2-recomp](https://github.com/AksharDP/xc2-recomp) — 移植版本体，及记录过场画面异常的
  [#58](https://github.com/AksharDP/xc2-recomp/issues/58) / [#46](https://github.com/AksharDP/xc2-recomp/issues/46)。
