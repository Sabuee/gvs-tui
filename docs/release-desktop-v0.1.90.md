# Desktop v0.1.90

配套 TUI **v0.2.87** / 网关 **v3.9.119**。桌面端与 TUI 共用下载库（`app/src/main/core.ts`
直接用 `@tui/quality.ts`、`@tui/jobs.ts`），所以 TUI v0.2.86 / v0.2.87 的国区改动在这一版一起生效。

## 变更（共用库，桌面端同步生效）

### 优酷：帧享两条帧率阶梯

网关 `tier=multi` 默认 `fps=all`，除 5 个 60fps 档还会追加 4 个 25 帧母版档
（`frame_xiang_dv25 / imax25 / hdr10_25 / sdr25`）。UPS 的 `max_fps` 是硬上限，
`profile=full/hdr10/sdr` 全声明 60/120，只回 `*_hfr_*` —— 客户端不 pin 帧率档就会出现
「选了 24 帧那一行、下载却拉成 60 帧」，随后按 `stream_type` 匹配找不到流。

- `src/lib/youku-play-input.ts`：按选中 `stream_type` 反推
  `dovi=dv25|dv60|hdr10_25|hdr10_60|sdr25|sdr60|imax25|imax60`（判据：有无 `_hfr_`），
  桌面端下载走 `@tui/jobs.ts` 的同一条 `playYouku`，自动带上；
- `src/lib/quality.ts`：画质探测显式 `fps: 'all'`；同分辨率同 HDR 偏好的撞名档位补
  「· 60帧 / · 24帧」，画质页能一眼分辨两条阶梯。

### 腾讯软字幕：清单带 URL，下载走 HLS 拼接

网关 v3.9.119 起 `subtitles[]` 带 `url` / `urls[]`，`url` 是 HLS 封装的 WebVTT
（`…<blob>.f53102000.vtt.m3u8?ver=4`）。桌面端目前只把字幕轨当作任务属性
（`subtitleTracks`），不落字幕文件；但清单解析修复后「有没有软字幕轨」的判定恢复正确，
画质页的字幕标记不再因缺 url 而漏判。

- 新增 `src/lib/vtt-hls.ts`：`parseHlsSubtitlePlaylist` / `stitchVttParts` / `vttToSrt` /
  `fetchHlsVtt`（与网关 `tencent/subtitle.go` 同口径）；
- `src/lib/online-subtitles.ts`：`OnlineSubtitle` 增 `playlist` / `urls`，
  playlist 字幕先拼 VTT 再落盘（TUI 侧使用；桌面端共用同一模块，后续接字幕下载即可复用）。

## 验证

- `bunx tsc --noEmit`（仓库根）无错误；`bun test src/lib` **1025 pass / 35 skip / 0 fail**；
- `app` 侧 typecheck 与打包由 Release 工作流完成（`desktop-v*` 标签触发 Windows / macOS 构建）。

## 兼容性

- 网关 < v3.9.119：`fps` / `dovi` 被忽略，优酷仍只列 60fps 档（等同 v0.1.89）；
- 网关 ≥ v3.9.119：两条帧率阶梯都能选、能下。

## 回滚

回退 `desktop-v0.1.89`。
