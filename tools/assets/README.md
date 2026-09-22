# TamaPoke 资源 —— 精灵图从何而来

精灵图不在此手动编辑或保存：它们由 `tools/` 中的脚本**从来源下载并打包**。
本文件夹只是流程文档（以前放着一份已废弃的“按 PNG 导入”方案）。

## 真实流程

microSD（`/mons/`）中并存两种格式，二者都派生自 PMD SpriteCollab：

| 格式 | 脚本 | 来源 | 是什么 |
|---|---|---|---|
| **TPK2** `pNNN.bin` / `psNNN.bin` | `pack_pmd.py` | [PMD SpriteCollab](https://github.com/PMDCollab/SpriteCollab)（CC BY-NC）| 多动作动画（待机、行走、睡觉、进食、受伤、攻击、手势）——用于**一切**：主界面与**图鉴 / 画廊** |
| **TPTH** `thumbs.bin` | `make_thumbs.py` | （派生自 TPK2）| 画廊的 40×40 缩略图 |

```bash
python3 tools/pack_pmd.py     # 151 + 异色 -> tools/sdcard/mons/p[s]NNN.bin
python3 tools/make_thumbs.py  # -> tools/sdcard/mons/thumbs.bin
python3 tools/send_sd.py      # 通过 USB 把全部内容发送到板子的 SD 卡
```

（`s` = 异色变体。`pack_pmd.py` 接受单个图鉴编号，
例如 `python3 tools/pack_pmd.py 7 25`。）

## 二进制格式

定义在各打包器的头文件中，并在 `sdmon.cpp` 中解析：

- **TPK1**（`SdMon`）：`"TPK1"`、`u16 w,h,frames,frameMs`、`u16 palCount`、
  `u16 pal[]`（RGB565）、`u8 data[frames*w*h]`（调色板索引，`0xFF` 透明）。
- **TPK2**（`PmdMon`）：`"TPK2"`、`u8 nActs`、`u16 palCount`、`u16 pal[]`，随后
  每个动作 `u8 id,w,h,nFrames` + `u16 ms[nFrames]` + `u8 data[w*h*nFrames]`。
- **TPTH**（`SdThumbs`）：`"TPTH"`、`u16 count`、`u32 offset[count]`，随后每个
  缩略图 `u8 w,h,palCount` + `u16 pal[]` + `u8 data[w*h]`。

固件在加载时会校验大小，因此被截断的 `.bin` 会被拒绝而不会破坏任何东西。

## 下载缓存

`pack_pmd.py` 会把 SpriteCollab 的原始 PNG 缓存到 `tools/pmd_cache/`
（被 git 忽略，可重新生成）。最终的 `.bin` 则版本化保存在
`tools/sdcard/mons/` 作为备份。

> 关于精灵图的来源与条款见 [CREDITS.md](../../CREDITS.md)。它们属于第三方：
> 请勿用于商业再分发。
