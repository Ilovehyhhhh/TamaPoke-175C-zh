# 致谢

TamaPoke 是一个**非商业、个人使用**的项目。它不销售、也不以商业方式再分发任何
受版权保护的材料。宝可梦及所有相关名称、设计和角色均为
**Nintendo / Game Freak / The Pokémon Company** 的商标与 ©。

本项目与上述公司无隶属关系，也未获其认可。

## 精灵图与数据

| 资源 | 来源 | 在项目中的用途 |
|---|---|---|
| **所有精灵图**（待机、行走、睡觉、进食、受伤、攻击……）| [PMD Sprite Collaboration (PMDCollab/SpriteCollab)](https://github.com/PMDCollab/SpriteCollab) | 处处使用的《不可思议的迷宫》风格动画精灵图：主界面、属性卡、小游戏，以及图鉴网格 + 详情视图 |
| **第一世代种族值** | [PokéAPI](https://pokeapi.co) | 每个物种真实的攻/防/速/HP |

**SpriteCollab** 精灵图是其社区画师们在各自条款下
（知识共享 署名-非商业性使用 4.0）的作品。按物种/按作者的署名见原仓库的
[tracker.json](https://github.com/PMDCollab/SpriteCollab/blob/master/tracker.json)。
衷心感谢整个社区付出的巨大努力。

> **如果你复用本仓库，请注意：** 打包的精灵图文件
> （`tools/sdcard/mons/*.bin`）派生自上述来源。请勿将其用于商业再分发。
> 如果你要发布本项目，干净的做法是**只分发代码与脚本**，让每位用户用
> `tools/pack_*.py`（或网页安装器）从原始来源下载并打包精灵图。

## 软件 / 硬件

| 组件 | 作者 / 来源 |
|---|---|
| GFX Library for Arduino | [moononournation](https://github.com/moononournation/Arduino_GFX) |
| SensorLib（CST9217 触摸、PCF85063 RTC）| [Lewis He / lewisxhe](https://github.com/lewisxhe/SensorLib) |
| XPowersLib（AXP2101 PMU）| [Lewis He / lewisxhe](https://github.com/lewisxhe/XPowersLib) |
| U8g2（仅中日韩字体数据）| [olikraus](https://github.com/olikraus/u8g2) |
| 开发板与引脚 | [Waveshare ESP32-S3-Touch-AMOLED-1.75](https://www.waveshare.com/wiki/ESP32-S3-Touch-AMOLED-1.75) |
| 网页安装器 | [ESP Web Tools](https://esphome.github.io/esp-web-tools/)（Nabu Casa）|

TamaPoke 自身的代码（固件与工具）均为原创作品。
