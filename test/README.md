# 测试

一套**在 PC 上、无需开发板**运行的测试：把固件的纯逻辑
（`pet.cpp`、`i18n.cpp`、`dex.h`）编译到一组 Arduino shim 上，并用假时钟和假
`random()` 执行，因此结果确定、运行不到一秒。

```bash
./test/run_tests.sh           # 全部
./test/run_tests.sh --asan    # 额外启用 AddressSanitizer + UBSan
```

或分步运行：

```bash
cd test && make               # 仅固件逻辑
cd test && make FILTER=evolve # 仅名称中含 “evolve” 的测试
cd test && make asan          # 带 sanitizers
python3 test/test_tools.py    # 仅 tools/ 中的工具
```

## 覆盖内容

| 文件 | 检查内容 |
|---|---|
| `test_pet.cpp` | 蛋与初始选择、进食、属性计时、睡眠、失误、等级、进化（含伊布分支）、战斗属性、训练与小游戏、连胜与羁绊、勋章、仪式、离线成长与 NVS 存档 |
| `test_dex.cpp` | 图鉴完整性：ASCII 名称、无环的进化链、所有物种可达、无无法完成的进化线、UI 缓冲区上限 |
| `test_i18n.cpp` | 9 种语言完整，非 CJK 语言无重音符号（GFX 字体为 ASCII），且 `%u`/`%s` 与 sketch 每次调用时传入的一致 |
| `test_tools.py` | `tools/` 的脚本可运行、图鉴数据自洽，且 `dex.h` 仍与 `tools/gen_dex.py` 生成的相符 |

## 如何搭建

- `shim/Arduino.h` + `shim/Preferences.h` + `shim/shim.cpp`：`pet.cpp` 和
  `i18n.cpp` 用到的最小 Arduino 子集。时钟（`millis()`）和 `random()` 是假的、
  可控的；`Preferences` 是一个模拟 NVS 的内存映射，因此可以通过创建另一个
  `Pet` 并调用 `begin()` 来模拟重启。
- `framework.h` + `main.cpp`：自研微型框架，无依赖。
- 标记为 `TEST_KNOWN_ISSUE` 的测试描述某个目前失败之处的**正确**行为：它们不会
  让测试套件失败，会单独列出，并在某天它们开始通过时提醒（以便移除标记）。

## 不覆盖的内容

所有需要开发板的部分：屏幕与触摸（`TamaPoke.ino`）、音频（`audio.cpp`）、
SD 卡（`sdmon.cpp`）以及 RTC/电池（`rtcbat.cpp`）。`.ino` 也不在此编译：需要带
ESP32 核心的 `arduino-cli`。
