# TamaPoke

[![在浏览器中刷机](https://img.shields.io/badge/flash-in%20browser-FF6B00?logo=googlechrome&logoColor=white)](https://socquique.github.io/TamaPoke/web/)
[![MakerWorld](https://img.shields.io/badge/MakerWorld-3D%20case-00AE42?logo=bambulab&logoColor=white)](https://makerworld.com/es/models/2937822-tamapoke-a-pokemon-pokeball-tamagotchi)
![Board](https://img.shields.io/badge/board-ESP32--S3%20round%20AMOLED-E7352C?logo=espressif&logoColor=white)
![Firmware](https://img.shields.io/badge/firmware-v1.18-8A2BE2)
[![CI](https://github.com/socquique/TamaPoke/actions/workflows/ci.yml/badge.svg)](https://github.com/socquique/TamaPoke/actions/workflows/ci.yml)
![Code](https://img.shields.io/badge/code-MIT-blue)
![Languages](https://img.shields.io/badge/languages-9-FFCB05)
[![Stars](https://img.shields.io/github/stars/socquique/TamaPoke?style=flat&logo=github&color=yellow)](https://github.com/socquique/TamaPoke/stargazers)

一款以第一世代宝可梦为灵感的电子宠物，运行在
**Waveshare ESP32-S3-Touch-AMOLED-1.75**（圆形 466×466 AMOLED，QSPI 接口的
CO5300 驱动，I2C 接口的 CST9217 触摸）。养育 151 只中的任意一只，让它进化、
训练，并把它们（含异色）全部集齐。

> **个人、非商业的同人项目。** 代码为 MIT；精灵图来自
> PMD SpriteCollab（CC BY-NC，宝可梦 © Nintendo/Game Freak），3D 外壳为
> CC BY-NC-SA。见 **[许可证](#许可证)** 与 **致谢**。

🔴 **3D 打印精灵球外壳 + 打印配置 → [在 MakerWorld 上](https://makerworld.com/es/models/2937822-tamapoke-a-pokemon-pokeball-tamagotchi)** · 在浏览器中刷机 → **[网页安装器](https://socquique.github.io/TamaPoke/web/)**

## 项目状态

已在真机运行。已实现：从 microSD 加载 151 只 + 异色的动画、完整生命周期
（按稀有度出蛋 → 进化 → 离别/放生/逃跑，每个都由决策对话框把关）、带图鉴的
繁育记录、战斗属性（基因 + 训练）、留存钩子（连胜 / 羁绊 / 勋章 / 昵称）、
生物群系 + 实时背景、精灵球小游戏、训练沙袋、泡泡洗澡动画、带离线成长的
RTC、电池管理（AXP2101）与 PWR 按键、防烧屏自动调暗、**声音（ES8311）**、
**9 种界面语言（默认简体中文）**、**首次运行的初始宝可梦选择**，以及一键
**网页安装器**。

待办：野外遭遇 / 战斗（已设计，未实现）、3D 外壳、耐久测试。见 **路线图**。

## 游戏手册（真实数值）

游戏实际运行方式的速查（数值直接取自代码）。

### 时间与升级
- **现实 1 分钟 = 游戏内 1 分钟。** 你的宝可梦每现实 **1 小时 +1 级**。升级
  完全基于时间——照顾得好不会加快，但照顾不周会*推迟进化*。
- **关机也会继续变老**（RTC 持续运行），最多补算 **2 周**。

### 四项属性（0–100）
需求：**食物**、**心情**、**体力**、**卫生**。初始 80 / 80 / 80 / 100。
**清醒**状态下，每分钟：

| 属性 | 每分钟下降 | 说明 |
|---|---|---|
| 食物 | −2 | |
| 体力 | −1 | 超重时额外 −1（体重 > 50 → 迟钝）|
| 卫生 | −1 | 屏幕上有便便时**每个再 −4**（最多 3 个）|
| 心情 | −1 | 食物 < 30 时额外 −2，卫生 < 30 时额外 −2 |

- 约 **15 %/分钟** 概率拉便便（仅当食物 > 40）。便便会迅速拉低卫生。
- **照顾失误** = 任一属性降到 **≤ 10**（30 分钟冷却，只计一次）。每次失误
  **推迟进化 1 级**，并冷却羁绊。

### 操作
- 🍎 **树果**（3 种口味）：+25 食物。每只宝可梦都有一个**隐藏的最爱口味**
  → +35 食物、+10 心情、♥、羁绊，并会被揭示。
- 🍬 **糖果：** +10 食物、+12 心情，但 **+12 体重**（会发胖）。
- ⚽ **玩耍 / 小游戏：** +心情、−体力；小游戏训练**速度**并燃烧体重。
- 🥊 **训练沙袋：** 训练**力量**（约 4 击 = 1 点，单次上限 +18），会消耗它。
- 🫧 **洗澡：** 清空便便，卫生 → 100。
- 👆 **抚摸它：** +5 心情 + 羁绊。
- 🌙 **睡觉：** 休息——体力 **+6/分钟**，需求下降约**慢 4 倍**且带下限
  （食物 30 / 心情 35 / 卫生 45）。不拉便便、不计失误、睡着时不会逃跑。

### 蛋与能孵出谁（出现概率）
- **第一个伙伴：** 你选择一只初始宝可梦——**妙蛙种子 / 小火龙 / 杰尼龟**。
- 孵蛋：点它 **3 次**（或等一会——它会自行孵化）。
- 之后每个蛋都会掷一个**稀有度档位**（在约 79 个可从蛋中获得的初始形态中）：

| 档位 | 基础概率 | 好好送别之后 | 数量 |
|---|---|---|---|
| ✨ 传说 | ~3 %\* | ~10 % | 5 |
| 🔵 稀有 | ~27 % | ~45 % | 27 |
| ⚪ 普通 | 其余 | 其余 | 47 |

  \* 只有在你**已注册 ≥ 25** 只宝可梦后，传说才会开始出现。
- 每日**连胜**和高**羁绊**会提高稀有/传说概率。
- 一次干净的**送别会祝福**下一个蛋；**逃跑会诅咒**它（强制普通）。
- 同一档位内会偏向**你尚未集齐的进化线**（因此 151 只都可完成）。
- **异色：** 基础 **1 / 48**（送别后立即变为 **1 / 24**），受连胜/羁绊提升，
  最好可达 **1 / 8**。在图鉴中单独记录。
- 每次孵化都会掷出独一无二的**基因**（每项属性 90–110 %）——没有两只完全相同。

### 进化
- 当**等级 ≥ 其进化等级**（大多数初始形态为 16；石系约 30，通讯系约 40）
  **且此刻每项属性 ≥ 40** 时触发。
- **绝不自动发生**——会出现一个按钮，**由你点击见证**（旧形态与新形态之间
  闪烁切换）。每次**失误推迟 1 级**。
- 你可以**拒绝**（“保持形态”）；下一级时会再次提供。
- *伊布*会根据你仍缺的进化分支来选择。

### 三种结局（每种都由你选择并见证——都不会自动触发）
- 💛 **离别**——当它已是**最终形态**且已生活 **3 天**。会出现一个按钮；
  触发它会**祝福你的下一个蛋**。你可以**推迟**（“继续陪伴”，一天后再次
  提供）。美好的结局。
- 💔 **逃跑**——如果你让**四项属性全部为 0 持续整整一小时**。任何一次照顾
  都会取消它。它会**诅咒下一个蛋**（强制普通）。悲伤的结局。
- 👋 **放生**——长按生物，按你的意愿送走它（中立）。

任何结局之后，都会出现一枚**新蛋**。

### 羁绊、连胜、勋章、图鉴
- **连胜**（属于玩家，跨宠物保留）：每个现实日的第一次照顾推进连胜；里程碑在
  **3 / 7 / 30 / 100** 天；跳过一天会中断。
- **羁绊**（属于宠物，孵化时重置）：随亲密度缓慢增长（**每日上限 +8**），
  疏于照顾会冷却。连胜与羁绊都会改善蛋/异色概率。
- **8 枚勋章**（Lv10/25/50、发现最爱树果、7 天连胜、满羁绊、最终形态、
  “健康” = 体重 0 且无失误），每只宠物一份 + 一个全局计数器。
- **图鉴：** 养育某个物种即注册它；集齐 **151 + 异色**。

### 战斗属性
攻 / 防 / 速 = 真实**第一世代种族值** × 基因 + 等级 + 训练（力量 ← 沙袋，
速度 ← 小游戏，防御 ← 连续 12 小时的良好照顾）。*（战斗：在路线图上。）*

## 硬件

- 开发板：[ESP32-S3-Touch-AMOLED-1.75](https://www.waveshare.com/wiki/ESP32-S3-Touch-AMOLED-1.75)
  —— 请选 **Standard**（无外壳）或 **-G**（带 GPS，也能装）版本；**不要选 “-B”**
  （自带保护壳，装不进去）。单独的 “1.75**C**” 是另一块板子。
  - **购买渠道**：[Waveshare 商店](https://www.waveshare.com/esp32-s3-touch-amoled-1.75.htm)（常缺货），或亚马逊上的 **-G**——TamaPoke 不使用 GPS，除此之外两块板子完全相同：🇺🇸 [.com](https://www.amazon.com/dp/B0F7XWWMJW?tag=capsuleradar-20) · 🇪🇸 [.es](https://www.amazon.es/dp/B0F7XWWMJW?tag=capsuleradar-21) · 🇩🇪 [.de](https://www.amazon.de/dp/B0F7XWWMJW?tag=capsulerada03-21) · 🇬🇧 [.co.uk](https://www.amazon.co.uk/dp/B0F7XWWMJW?tag=capsulerada0d-21) · 🇮🇹 [.it](https://www.amazon.it/dp/B0F7XWWMJW?tag=capsulerada08-21) · 🇫🇷 [.fr](https://www.amazon.fr/dp/B0F7XWWMJW?tag=capsulerada0e-21) · 🇯🇵 [.co.jp](https://www.amazon.co.jp/dp/B0F7XWWMJW?tag=capsuleradar-22) · 🇨🇦 [.ca](https://www.amazon.ca/dp/B0F7XWWMJW?tag=capsulerada09-20)。可选电池——1100 mAh 带保护锂电池，MX1.25 插头：🇪🇸 [.es](https://www.amazon.es/dp/B0F1FGZQS5?tag=capsuleradar-21) · 🇺🇸 [.com](https://www.amazon.com/dp/B0F1FGZQS5?tag=capsuleradar-20) · 🇮🇹 [.it](https://www.amazon.it/dp/B0F1FGZQS5?tag=capsulerada08-21) · 🇫🇷 [.fr](https://www.amazon.fr/dp/B0F1FGZQS5?tag=capsulerada0e-21)（DE/UK：该型号未上架——请搜索*带保护*的 3.7 V 锂电池，约 1100 mAh，尺寸 **102540**，1.25 mm micro-JST 插头）——接板前请先核对插头极性。<sub>亚马逊链接为联盟链接；作为 Amazon Associate，我从符合条件的购买中获利。</sub>
  - **microSD 卡**（存放精灵图集——任意小的 class-10 卡即可）：🇪🇸 [.es](https://www.amazon.es/s?k=microsd+32gb&tag=capsuleradar-21) · 🇺🇸 [.com](https://www.amazon.com/s?k=microsd+32gb&tag=capsuleradar-20) · 🇩🇪 [.de](https://www.amazon.de/s?k=microsd+32gb&tag=capsulerada03-21) · 🇬🇧 [.co.uk](https://www.amazon.co.uk/s?k=microsd+32gb&tag=capsulerada0d-21) · 🇮🇹 [.it](https://www.amazon.it/s?k=microsd+32gb&tag=capsulerada08-21) · 🇫🇷 [.fr](https://www.amazon.fr/s?k=microsd+32gb&tag=capsulerada0e-21) · 🇯🇵 [.co.jp](https://www.amazon.co.jp/s?k=microsd+32gb&tag=capsuleradar-22) · 🇨🇦 [.ca](https://www.amazon.ca/s?k=microsd+32gb&tag=capsulerada09-20)
- 圆形 466×466 AMOLED，**CO5300** 驱动（QSPI，80 MHz）
- 电容触摸 **CST9217**（I2C，地址 0x5A）
- **AXP2101**（电源管理 + 电池 + PWR 按键）、**PCF85063**（RTC）、
  microSD 卡槽、**ES8311** 音频编解码（→ 功放 → MX1.25 接口上的外接扬声器）
- 引脚取自[官方 Waveshare 仓库](https://github.com/waveshareteam/ESP32-S3-Touch-AMOLED-1.75)（见 `pin_config.h`）

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
arduino-cli compile --fqbn "$FQBN" .
arduino-cli upload -p /dev/cu.usbmodemXXXX --fqbn "$FQBN" .
```

### 最省事的安装：网页安装器

`web/index.html` 会刷入固件（ESP Web Tools），并通过 Web Serial 把精灵图推送到
SD 卡，无需 Arduino。请通过 HTTPS 或 `localhost`（安全上下文）提供它，并在
**Chrome/Edge** 中打开。见 [`web/README.md`](web/README.md)。

### 自行生成并加载精灵图

所有精灵图都来自 **[PMD SpriteCollab](https://github.com/PMDCollab/SpriteCollab)**
（CC BY-NC）。你可以用下面的流水线重新生成整套图并加载到板子上——固件通过 USB
接收文件（带逐块 ACK 的 PUT 协议），因此无需取下卡（必要时它会将 SD 格式化为
FAT）。

```bash
python3 tools/pack_pmd.py       # 抓取 + 打包 PMD 精灵图：151 + 异色 -> tools/sdcard/mons/p[s]NNN.bin
python3 tools/make_thumbs.py    # 图鉴缩略图（由 PMD 精灵图生成）-> thumbs.bin
python3 tools/send_sd.py        # 通过 USB 把 tools/sdcard/mons/* 发送到板子的 SD 卡
```

若想制作**一键网页安装器包**而非通过 USB 发送：

```bash
python3 tools/pack_bundle.py    # 将 tools/sdcard/mons/* 打包进 web/sprites.pak
```

然后在网页安装器的 **“Load sprites”** 按钮中加载它（或使用上面的 `send_sd.py`）。
`pack_pmd.py` 也接受单个图鉴编号，如 `pack_pmd.py 7 25`。
（总计约 40 MB，全部为 PMD。版本化保存于 `tools/sdcard/` 下。）

## 如何游玩

首次运行时，你会**选择一只初始宝可梦**（妙蛙种子 / 小火龙 / 杰尼龟）。之后你
会从一枚**蛋**开始。点它 3 次或等一会，它就会孵化。从那时起，照顾你的伙伴：

**四项会衰减的属性**：**食物**、**心情**、**体力**、**卫生**。若有一项触底，
就算一次*失误*。

**按钮（底部弧形，图标）：**
- 🍎 **喂食** → 食物菜单：3 种树果（每只物种都有一个带来加成的隐藏最爱）
  和一颗糖果（+心情但会发胖；体重会让它迟钝）。
- ⚽ **玩耍** → 精灵球小游戏（训练速度）。
- 🌙 **灯光** → 睡眠/唤醒（恢复体力，调暗屏幕）。睡着时，需求下降慢得多
  （休息）。
- 🫧 **洗澡** → 一段泡沫场景，清理便便。

**触摸手势：**
- 点生物 = 抚摸它（+心情、羁绊）。
- 水平滑动 = 打开**图鉴 / 画廊**。
- 垂直上滑 = 打开**属性卡**（4 页：资料 / 战斗 / 勋章 / 成长；在它们之间滑动；
  在资料页点名字可改名；在战斗页“训练力量”按钮会打开沙袋）。
- 下滑 = **设置时钟**并选择**语言** + 声音开关。
- 长按生物（3 秒）= **放生**对话框。

**物理 PWR 按键：** 短按 = 屏幕开/关 · 长按（4 秒）= 完全关机
（RTC 保持运行，因此关机时时间也会流逝）。

## 决策：你来选择，也由你见证

三种生命周期结局和进化**不会自行发生**——条件满足时会出现一个按钮，由你点击
（这样你就能在场见证），每个都会打开一个二选一对话框：

- **进化**（红色按钮）：*进化*（史诗动画：光环、光线、闪光，以及**旧形态与
  新形态之间的闪烁**）或*保持形态*（下一级再次提供）。
- **离别**（金色按钮，最终形态 + 3 天）：*告别*（温暖的告别，升起的爱心 →
  新蛋）或*继续陪伴*（留住你的伙伴；一天后再次提供）。张力：一只养到极致的
  伙伴 vs. 集齐图鉴。
- **逃跑**（深色按钮，完全忽视 1 小时）：一个在雨中“觉得被抛弃”的悲伤结局
  ——照顾它即可取消。

## 精灵图：处处皆 PMD SpriteCollab

- **PMD SpriteCollab**（全部——主界面、属性卡、小游戏**以及图鉴网格 + 详情
  视图**）：行为精灵图——`tools/pack_pmd.py` 把动作（Idle、Walk L/R、Sleep、
  Eat、Hurt、Attack、Pose、Nod、DeepBreath）打包成多动作的 **TPK2** 格式
  （`/mons/pNNN.bin`）。`TamaPoke.ino` 中的引擎让生物漫步、做手势、蜷缩睡觉、
  咀嚼和皱眉。以脚部（最低的内容行）为锚点，而非画布。图鉴缩略图
  （`thumbs.bin`，TPTH）由 `tools/make_thumbs.py` 从这些图派生。
- **自研工坊**（`tools/sprites.py`）：9 个手绘精灵图作为无 SD 卡时的回退 +
  UI 图标。生成 `species.h`。预览见 `tools/sheet.png`，用
  `python3 tools/sprites.py emit` 输出。

`sdmon.h/.cpp` 把 PMD 精灵图加载进 PSRAM（TPK2 用 `PmdMon`）以及缩略图
（`SdThumbs`）。`SdMon`（TPK1）仅作为休眠的旧版回退保留。

## 图鉴与物种数据

`tools/dex_data.py` 是**唯一数据源**：名称、slug、属性（强调色 + 背景生物群系）、
带第一世代等级的进化线、稀有度和初始宝可梦。`tools/dex_stats.py` 存放真实的
种族值（来自 PokéAPI）。`tools/gen_names.py` 从 PokéAPI 拉取**官方本地化名称**
写入 `tools/dex_names.py`（第一世代只有法语和德语不同；西班牙语、意大利语和
葡萄牙语使用英文名）。`gen_dex.py` 生成 `dex.h`（`DEX_TBL[152]` 表加上各语言
名称表和 `dexName()` 访问器）。宠物的身份就是它的图鉴编号（持久化在 NVS）。

- **进化**为第一世代风格（16/36 级……；石系约 30，通讯系约 40；伊布会分支到
  你仍缺的那个进化）。每次失误推迟 1 级；任一属性 < 40 或睡着时不会进化。

## 战斗属性与训练

每只生物的攻/防/速 = 真实第一世代种族值 × **基因**（90–110 %，孵化时掷出）+
等级 + **训练**：
- 速度 ← 小游戏
- 防御 ← 持续的良好照顾（12 小时无失误）
- 力量 ← 训练沙袋（击打）

显示在属性卡的战斗页。隐藏的体重会随糖果上升，并随训练燃烧掉。

## 留存：连胜、羁绊、勋章、昵称

- **连胜**（玩家的，跨生物保留）：每个现实日的第一次照顾推进连胜；3/7/30/100
  里程碑会庆祝；跳过一天会中断。主界面有火焰徽章。
- **羁绊**（生物的）：随照顾和抚摸缓慢上升，随失误下降。
- **勋章**（个体：等级、树果、连胜、羁绊、最终形态、健康）+ 一个全局计数器。
  属性卡的勋章页。
- **昵称**：触摸键盘；昵称决定标题栏和卡片上的显示。

高连胜与高羁绊会**改善蛋的掷取**（稀有度与异色）：用心照顾总有回报。

## 生命周期、按稀有度出蛋、语言

生命周期持续 **3 天**游玩。三种结局（都会留下新蛋）：
**离别**（最终形态 + 3 天）、**放生**（长按）、**逃跑**（四项条为 0 持续 1 小时）。
每个繁育过的物种都会记录在**繁育图鉴**中（普通与异色分开记录）。

蛋在约 79 个初始形态（47 普通 / 27 稀有 / 5 传说）中掷稀有度，
**偏向你尚未集齐的进化线**（151 只都可完成），受离别祝福、受逃跑诅咒。
只有注册 25+ 只后才会出传说。
**异色** 1/48（连胜/羁绊/离别会更高）。

**语言：** 界面支持 9 种语言——简体中文（默认）、英语、西班牙语、法语、德语、
意大利语、葡萄牙语、日语和韩语——可在设置界面（下滑）切换。
**宝可梦名称也已本地化**：法语、德语、日语、韩语和中文显示官方名称
（Bulbizarre、Bisasam、フシギダネ、이상해씨、小火龙……）。西班牙语、意大利语和
葡萄牙语使用英文名，这是这些地区第一世代的官方用法。尼多兰♀/♂ 在日语中读作
ニドランF/M，在韩语中读作 니드런 암/수，在中文中为 尼多兰/尼多朗：没有任何 U8g2
unifont 子集带有 ♀ 或 ♂，因此每种语言各选自己的替代。

## 背景：生物群系 + 实时

待机界面根据 **RTC 的真实时间**绘制天空（黎明 / 白天 / 黄昏 / 夜晚，含月亮和
星星），并根据**属性的生物群系**绘制地面（草原、海滩、森林、火山、山地、雪地）。
睡眠会强制为夜晚。

## 目录结构

- `TamaPoke.ino` —— 初始化、游戏循环、各界面渲染、手势、串口控制台、音频
- `pet.h` / `pet.cpp` —— 宠物状态与逻辑（属性、进化、生命周期、连胜/羁绊/勋章、NVS）
- `sdmon.h` / `sdmon.cpp` —— TPK1（动画）与 TPK2（PMD）精灵图 + 缩略图，以及通过 USB 接收文件（PUT/LS）
- `rtcbat.h` / `rtcbat.cpp` —— PCF85063 RTC + AXP2101 PMU（电池、亮度、PWR 按键）
- `audio.h` / `audio.cpp` —— ES8311 + I2S + Game-Boy 风格音调合成（非阻塞任务）
- `i18n.h` / `i18n.cpp` —— 9 种语言的字符串表
- `dex.h` —— 生成文件（`gen_dex.py`）：151 表
- `species.h` —— 生成文件（`sprites.py`）：回退精灵图、UI 图标、颜色
- `pin_config.h` —— 板子的官方引脚
- `tools/` —— 流水线：`dex_data.py`（数据）、`dex_stats.py`、`dex_names.py` +
  `gen_names.py`（本地化名称）、`gen_dex.py`、
  `sprites.py`（工坊）、`pack_pmd.py` / `make_thumbs.py`
  （打包器）、`pack_bundle.py`（网页包）、`send_sd.py`（SD 上传）、`touch_log.py`
- `tools/sdcard/mons/` —— 生成的 .bin 文件（动画、异色、PMD、缩略图）
- `web/` —— 浏览器安装器（ESP Web Tools + Web Serial 精灵图加载器）

## 串口控制台（115200，调试）

`STATS`（完整状态）· `SPEC <dex>`（切换物种）· `LVL <n>` · `HATCH` ·
`SHINY` · `NICK <x>` · `BYE` / `RUN`（离别 / 逃跑）· `ABANDON`（强制进入可逃跑
状态）· `WIPE`（恢复出厂 → 新游戏）· `BEEP`（音频测试）·
`REG`（图鉴）· `EGGS`（模拟 20 个蛋）· `GAL`（画廊）· `CAREDAY` ·
`TIME <epoch>` / `RTCSET <epoch>` · `HEALTH`（运行时长 + 堆内存，用于耐久测试）·
`LS` / `PUT`（SD 文件）。

快速测试：调低 `pet.h` 中的 `PET_TICK_MS`、`MINUTES_PER_LEVEL` 和 `FAREWELL_AGE_MIN`。

## 路线图

- **野外遭遇 / 战斗** —— 已设计（见项目记忆）：按攻/防/速结算，配合 PMD 的
  Attack/Hurt 动画，训练师等级作为终局内容。风格仍待定（自动 / 时机 / 回合制）。
- **耐久测试** 24–48 小时（已具备检测手段：`HEALTH` 命令/心跳）。

*（已完成：3D 打印外壳[发布于 MakerWorld](https://makerworld.com/es/models/2937822-tamapoke-a-pokemon-pokeball-tamagotchi)；仓库公开，含浏览器安装器 + 一键精灵图包。）*

## 社区分支

- **[TamaPoke — Expanded](https://github.com/ShadowEnemyx/TamaPoke/tree/tamapoke-expanded-update)**，作者 **ShadowEnemy** —— 一个内容丰富的社区分支（不同作者/分支）：完整的**属性相克战斗系统**、全部 **151 + 异色**并带**图鉴 / 收藏盒**与每日目标、**6 种界面语言**、**ES8311 声音**、初始选择和一键网页安装器。值得一看。🎮
- **[TamaPoke](https://github.com/DylanPDao/TamaPoke)**，作者 **DylanPDao** —— 另一个内容丰富的分支：**道馆战**和两台设备间的**局域网对战**、**招式表**、**队伍 + 盒子**系统、**EV/IV** 属性系统，且覆盖范围扩展到**第三世代（386）**。沿用 PMD 精灵图流水线。🏆

## 致谢

所有精灵图：[PMD SpriteCollab](https://github.com/PMDCollab/SpriteCollab)
（社区，CC BY-NC）。种族值：[PokéAPI](https://pokeapi.co)。宝可梦是
Nintendo / Game Freak / The Pokémon Company 的 ™。非商业、个人使用项目。
完整列表见 [`CREDITS.md`](CREDITS.md)。版本历史见
[`CHANGELOG.md`](CHANGELOG.md) —— 其中的多数修复都来自亲手制作并反馈问题的
人们。

## 许可证

- **源代码**（固件 + 工具）：**[MIT](LICENSE)**。
- **精灵图与名称**：© Nintendo / Game Freak / The Pokémon Company；像素画
  来自 [PMD SpriteCollab](https://github.com/PMDCollab/SpriteCollab)（CC BY-NC 4.0）。
  **仅限非商业使用。**
- **3D 打印外壳**：为 **yoyothechicken** 的 *“Pokeball”* 的二次创作
  （[MakerWorld #839922](https://makerworld.com/es/models/839922-pokeball)），
  以 **CC BY-NC-SA** 授权，在此以相同条款分享。

这是一个非官方同人项目，与 Nintendo 无隶属关系，也未获其认可。
