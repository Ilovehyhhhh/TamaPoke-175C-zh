# TamaPoke 网页安装器

一个一键页面，在浏览器中（Chrome/Edge）刷入固件并加载精灵图，无需 Arduino 或
驱动。它使用 [ESP Web Tools](https://esphome.github.io/esp-web-tools/) 刷机，
并用 **Web Serial** 通过固件的 `PUT` 协议（与 `tools/send_sd.py` 相同）把精灵图
推送到 SD 卡。

## 内容

- `index.html` —— 页面（刷机 + 精灵图加载器）。
- `manifest.json` —— ESP Web Tools 配置（指向固件）。
- `firmware/tamapoke.bin` —— 合并后的固件，可在 `0x0` 处刷入。
- `sprites.pak` —— 一个包内的全部精灵图（TPAK），页面可一键发送。
  由 `tools/pack_bundle.py` **生成**（默认被 git 忽略——见下文*托管精灵图*）。

## 重新生成

修改固件或精灵图之后：

```bash
bash tools/build_web.sh        # 重新编译 -> firmware/tamapoke.bin 并重建 sprites.pak
```

## 本地测试

Web Serial 与 ESP Web Tools 需要**安全上下文**：`https://` 或
`http://localhost`。测试：

```bash
cd web && python3 -m http.server 8000
# 在 Chrome/Edge 中打开 http://localhost:8000
```

## 终端用户流程

1. **安装 TamaPoke** → 刷入固件（选择 USB 端口；新板子请勾选 “Erase
   device”）。
2. **连接开发板** + **加载精灵图** → 下载 `sprites.pak` 并通过 USB 复制到
   microSD（有进度条，约 8–10 分钟）。请先关闭第 1 步的安装标签页：同一时间
   只能有一个程序使用该端口。
3. 重启（PWR 按键）→ 选择你的初始宝可梦并开始游玩。

一个隐藏的“手动挑选”选项让进阶用户发送自己的 `.bin`。

## 托管精灵图

`sprites.pak` 约 58 MB 且被 **git 忽略**，以免撑大仓库。要让一键精灵图加载器在
真实部署中可用，选一种：

- **提交它** —— 把 `web/sprites.pak` 加入 git 并从 Pages 提供。简单，
  但会使仓库的精灵图体积翻倍。
- **发布资源** —— 把 `sprites.pak` 附到 GitHub Release，并把 `index.html` 中的
  `fetch('sprites.pak')` URL 改为 release URL（保持仓库小巧；注意资源主机的
  CORS）。

所有精灵图均来自 PMD SpriteCollab，CC BY-NC（允许带署名的非商业分享）；
见 [`../CREDITS.md`](../CREDITS.md)。

## 部署（GitHub Pages）

1. 仓库设置 → Pages → 从 `main` 提供，目录 `/web`（或把 `web/` 移到
   `docs/`）。Pages 会自动提供 HTTPS。
2. 最终 URL 为 `https://<user>.github.io/<repo>/`。

> **私有仓库使用 Pages** 需要 GitHub Pro/Team。如果你为了免费使用 Pages 而把
> 仓库设为**公开**，请先决定精灵图的处理方式（见上文与 CREDITS）。

## 限制

- 仅桌面版 **Chrome/Edge**（Firefox/Safari 不支持 Web Serial）。
