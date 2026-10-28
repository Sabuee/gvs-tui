# TUI v0.2.87

配套网关 **v3.9.119**。v0.2.86 只改了字幕一侧，漏了优酷：网关现在一次给两条帧率阶梯
（60fps 的 `*_hfr_*` 与 25/24 帧母版），客户端不 pin 帧率档就会「选了 24 帧那一行、
下载却拉成 60 帧」，随后按 `stream_type` 匹配找不到流。本次补上。

## 变更

### 优酷：选档 pin 帧率档（`src/lib/youku-play-input.ts`）

- 按选中的 `stream_type` 反推 getvideo 档位名，给 `play` 带 `dovi=`：
  `dv25/dv60`、`hdr10_25/hdr10_60`、`sdr25/sdr60`、`imax25/imax60`
  （判据：stream_type 里有没有 `_hfr_`）；
- 原因：UPS 的 `max_fps` 是硬上限，`profile=full/hdr10/sdr` 都声明 60/120，
  只回 `*_hfr_*`，25/24 帧母版档必须显式给档位名才下发；
- `profile` 保留（旧网关只认它），新网关由 `dovi` 覆盖 HDR/SDR 偏好与帧率上限；
- TV / App 端不加 `dovi`（服务端自己定帧率）。

### 优酷：探测显式要全档（`src/lib/quality.ts`）

- `probeOptions` 的 youku play 输入加 `fps: 'all'`，意图明确；
  旧网关忽略未知参数，行为不变。

### 优酷：撞名档位补帧率

- 同分辨率、同 HDR 偏好的两条阶梯 label 完全相同（都是「帧享4K 杜比」），
  撞名时补成「· 60帧 / · 24帧」，选择列表能一眼分辨。

## 验证

- `bunx tsc --noEmit` 无错误；
- `bun test src/lib/youku-play-input.test.ts`：**21 pass / 0 fail**
  （8 个档位名的 pin、TV/App 不 pin、探测带 fps=all、撞名补帧率）；
- `bun test src/lib`：**1025 pass / 35 skip / 0 fail**（138 个文件 / 1060 个用例）。

## 已知未覆盖

- 优酷 `fps=120` 档仍取决于片源是否真有 120 帧（服务端裁剪，网关不补帧）。
- 腾讯软字幕的真轨仍需会员 TV 会话；拼接逻辑由离线用例覆盖。

## 回滚

回退 `v0.2.86`（优酷选 25 帧档会退回 60fps 流或匹配失败）。
