# TamaPoke

[![在浏览器中刷机](https://img.shields.io/badge/flash-in%20browser-FF6B00?logo=googlechrome&logoColor=white)](https://shadowenemyx.github.io/TamaPoke/web/)
[![MakerWorld](https://img.shields.io/badge/MakerWorld-3D%20case-00AE42?logo=bambulab&logoColor=white)](https://makerworld.com/es/models/2937822-tamapoke-a-pokemon-pokeball-tamagotchi)
![Board](https://img.shields.io/badge/board-ESP32--S3%20round%20AMOLED-E7352C?logo=espressif&logoColor=white)
![Firmware](https://img.shields.io/badge/firmware-v1.36.1-4F93C4)
![Pokémon](https://img.shields.io/badge/Pok%C3%A9mon-386-E8503A)
![Languages](https://img.shields.io/badge/languages-7-FFCB05)

一款以宝可梦为灵感的电子宠物，运行在 **Waveshare ESP32-S3-Touch-AMOLED-1.75**
（圆形 466×466 AMOLED，QSPI 接口的 CO5300 驱动，I2C 接口的 CST9217 触摸）。
这是 **ShadowEnemyx 扩展分支**的中文移植版：涵盖**前三世代全部 386 只宝可梦**
（关都 / 城都 / 丰缘，含异色），并完整支持**简体中文**界面与宝可梦名称。

> **个人、非商业的同人项目。** 代码为 MIT；精灵图来自 PMD SpriteCollab
> （CC BY-NC，宝可梦 © Nintendo/Game Freak），3D 外壳为 CC BY-NC-SA。
> 见 **[许可证](#许可证)** 与 **[致谢](#致谢)**。

🔴 **3D 打印精灵球外壳 + 打印配置 → [在 MakerWorld 上](https://makerworld.com/es/models/2937822-tamapoke-a-pokemon-pokeball-tamagotchi)** · 在浏览器中刷机 → **[网页安装器](https://shadowenemyx.github.io/TamaPoke/web/)**

## 项目状态

本仓库在 **ShadowEnemyx/TamaPoke**（扩展分支，固件 1.36.1）基础上加入了**简体中文**，
并作为默认语言。已实现：

- **386 只宝可梦**（关都 + 城都 + 丰缘），含异色变体
- **地区 / 初始宝可梦选择**（新存档时）
- 完整的**属性相克战斗系统**（野外遭遇、攻击/闪避/休息、捕捉）
- **5 个小游戏**：精灵球、捕捉、记忆、清洁、类型；每日目标
- **远征**系统，带一个持久化的小型道具背包
- 档案、性格、勋章、收藏家等级与外观边框
- RTC 昼夜循环 + 离线成长
- 合成物种叫声与音效（无游戏音频 / ROM 采样）
- 常驻**计步器**（左上角 + 步数/路线卡片）
- 浏览器**存档备份 / 恢复**，带自动安全备份
- 图鉴筛选（地区 / 养育 / 捕捉 / 异色）
- 行走奖励（每日 500 / 2000 / 5000 步，路线里程碑）

### 行走奖励

行走在熄屏、亮屏、甚至连接 USB 时都计数（由节奏过滤器确认真实步伐，单次抖动会被
忽略）。

- 每日奖励：**500 / 2000 / 5000** 步
- 终身路线等级：**10000 / 50000 / 100000 / 250000** 步
- 行走会缓慢提升**心情**与**亲密**（有每日/每小时上限）
- 每日与终身步数会略微提升野外异色概率与捕捉概率

## 基本操作

- 点宝可梦：抚摸它并播放合成叫声。
- 左右滑动：图鉴 / 画廊。
- 上滑：档案、性格、每日、盒子、战斗、勋章、成长、远征与步数/路线卡片。
- 下滑：时钟、语言、声音、省电与帮助。
- 长按宝可梦 3 秒：放生对话框。
- PWR 短按：屏幕开/关。PWR 长按：完全关机。

四项需求随时间衰减：**食物、心情、体力、卫生**。进化需要物种等级，且四项中至少
三项严格高于 40。进化、离别、放生与逃跑始终以决策对话框呈现，不会在后台悄悄完成。

## 硬件

- Waveshare ESP32-S3-Touch-AMOLED-1.75 **Standard** 或 **-G**（请勿选 **-B**；单独的
  **1.75C** 是另一块板子）
- 圆形 466×466 AMOLED，**CO5300** 驱动（QSPI，80 MHz）
- 电容触摸 **CST9217**（I2C，地址 0x5A）
- **AXP2101**（电源管理 + 电池 + PWR 按键）、**PCF85063**（RTC）、
  microSD 卡槽、**ES8311** 音频编解码（→ 功放 → MX1.25 接口上的外接扬声器）
- 引脚取自[官方 Waveshare 仓库](https://github.com/waveshareteam/ESP32-S3-Touch-AMOLED-1.75)（见 `pin_config.h`）
>>>>>>>>> Temporary merge branch 2

## 库（Arduino IDE / arduino-cli）

| 库 | 作者 | 用途 |
|---|---|---|
| GFX Library for Arduino（`Arduino_GFX`）| moononournation | QSPI 下的 CO5300 + PSRAM 中的帧缓冲 |
| SensorLib | Lewis He | CST9217 触摸 + PCF85063 RTC |
| XPowersLib | Lewis He | AXP2101 PMU（电池、亮度、PWR 按键）|
| U8g2 | olikraus | 中日韩字形（`unifont_t_japanese3`，唯一同时带 `！？。、「」` 的日语子集；`unifont_t_korean2` 用于谚文，2350 个音节，对比 korean1 的 478 个；中文用 `unifont_t_chinese3`）；只使用字体数据，不用其显示驱动 |
| ESP_I2S（ESP32 核心内置）| Espressif | 到 ES8311 编解码的 I2S |

## IDE 配置 / 编译

- 开发板：**ESP32S3 Dev Module** · Flash **16MB** · PSRAM **OPI PSRAM**
  （必需：466×466×16 位帧缓冲约 434 KB，位于 PSRAM 中）·
  带 FAT 的分区方案（如 `16M Flash (3MB APP/9MB FATFS)`）·
  USB CDC On Boot **Enabled**

```bash
FQBN="esp32:esp32:esp32s3:CDCOnBoot=cdc,FlashSize=16M,PSRAM=opi,PartitionScheme=app3M_fat9M_16MB"
```

验证过的工具链：ESP32 Arduino core `3.3.10`、GFX Library for Arduino `1.6.7`、
SensorLib `0.4.1`、XPowersLib `0.3.3`，以及用于中文字形的 **U8g2**（`unifont_t_chinese3`）。

构建并校验（不覆盖安装器产物）：

```bash
bash tools/build_web.sh --check            # 当前公开 Gen 3，386 只
bash tools/build_web_local.sh --check      # Gen 3 调试构建，386 只
bash tools/build_web_gen2_legacy.sh --check  # 历史 Gen 2 校验
```

构建本地产物、提供目录并打开所需页面：

```bash
bash tools/build_web_local.sh              # 或: bash tools/build_web_gen3.sh
python3 -m http.server 8000 --directory web
```

- `http://127.0.0.1:8000/` — 当前公开 Gen-3 安装器
- `http://127.0.0.1:8000/dev.html` — Gen-3 硬件/调试安装器
- `http://127.0.0.1:8000/release-gen3.html` — 本地发布参考页

**更新存档时切勿勾选 Erase device**。

### 精灵图流水线

精灵图来自 [PMD SpriteCollab](https://github.com/PMDCollab/SpriteCollab)，用仓库工具
生成与传输：

```bash
python3 tools/pack_pmd.py
python3 tools/make_thumbs.py
python3 tools/pack_bundle.py
python3 tools/pack_bundle.py --gen3
python3 tools/send_sd.py
```

## 测试

```bash
cd tests && make clean && make test
cd ..
bash tools/build_web.sh --check
bash tools/build_web_local.sh --check
bash tools/build_web_gen2_legacy.sh --check
git diff --check
```

硬件仍是显示裁剪、触摸边缘、IMU、扬声器音量、microSD 传输与存档保留的最终裁判。

## 本地化（中文）

- 界面语言：**7 种**（西班牙语、英语、法语、德语、意大利语、葡萄牙语、**简体中文**），
  **默认简体中文**。
- 386 只宝可梦名称全部使用**官方简体中文名**（来自 PokéAPI `zh-Hans`）。
- 中文使用 U8g2 `unifont_t_chinese3` 字形（UTF-8）。文本测量集中在 `textW()/centerX()`，
  光标补偿集中在 `setCur()/applyLangFont()`，因此六个拉丁语系语言不受影响。
- 图鉴与名称数据由 `tools/gen_dex.py` 从 `tools/dex_data.py` 生成（`LOCALIZED_NAMES`
  的第 7 列为中文名）。

## 致谢

- 原始项目：[socquique/TamaPoke](https://github.com/socquique/TamaPoke)
- 扩展分支与安装器：[ShadowEnemyx/TamaPoke](https://github.com/ShadowEnemyx/TamaPoke)
- 动态像素画：[PMD SpriteCollab](https://github.com/PMDCollab/SpriteCollab)，CC BY-NC 4.0
- 3D 精灵球外壳：[MakerWorld](https://makerworld.com/es/models/2937822-tamapoke-a-pokemon-pokeball-tamagotchi)，CC BY-NC-SA
- 宝可梦数据（种族值、类型、本地化名称）：[PokéAPI](https://pokeapi.co)

完整归属见 [`CREDITS.md`](CREDITS.md)。

## 许可证

- **源代码**（固件 + 工具）：**[MIT](LICENSE)**。
- **精灵图与名称**：© Nintendo / Game Freak / The Pokémon Company；像素画来自
  [PMD SpriteCollab](https://github.com/PMDCollab/SpriteCollab)（CC BY-NC 4.0）。
  **仅限非商业使用。**
- **3D 打印外壳**：yoyothechicken 的 *“Pokeball”* 二次创作
  （[MakerWorld #839922](https://makerworld.com/es/models/839922-pokeball)），CC BY-NC-SA。

这是一个非官方同人项目，与 Nintendo、Game Freak 或 The Pokémon Company 无隶属关系，
也未获其认可。
