# TUI v0.2.86

配套网关 **v3.9.119**（优酷 play 全档位 + 腾讯软字幕可下载）。本次只改客户端字幕一侧：
腾讯软字幕给的 `url` 是 **HLS 封装的 WebVTT**（`…f53102000.vtt.m3u8`），直接把 playlist 写盘
会得到「内容是 m3u8 的 .vtt 文件」，播放器整轨读不出来。

## 变更

### 字幕清单（`src/lib/online-subtitles.ts`）

- 网关 `subtitles[]` 现在带 `url` / `urls[]`（腾讯 4 条 CDN 镜像），下载器不再把腾讯行整批过滤；
- `OnlineSubtitle` 增可选 `playlist` / `urls`：`playlist` 由网关字段或 URL 后缀（`.m3u8`）判定；
- `label` 回退顺序补上 `name`（腾讯给的是 `name`，此前落成 id 做文件名）。

### HLS(WebVTT) 拼接（新增 `src/lib/vtt-hls.ts`）

- `parseHlsSubtitlePlaylist`：`#EXTINF` 累加出分片起点，相对 URI 按 playlist 解析；
  master playlist 退化为第一个 variant；
- `stitchVttParts`：cue 时间小于分片起点（相对时间）按起点平移、已大于等于起点的原样保留；
  `WEBVTT` 头只留一个，NOTE/STYLE/REGION 丢弃，重复 cue 去重 —— 与网关
  `internal/provider/tencent/subtitle.go` 同口径，客户端自取或网关代取结果一致；
- `vttToSrt`：时间戳逗号 + 序号 + 去 cue settings；
- `fetchHlsVtt`：走注入的 `fetcher`（和视频同一条出口），分片失败不整轨报废，一个 cue 都没有才报错。

### 下载路径

`downloadSubtitles` 对 `playlist` 字幕先拼 VTT 再落盘；普通直链（Crunchyroll 的 `.ass` 等）行为不变。

## 验证

- `bunx tsc --noEmit` 无错误；
- `bun test src/lib/vtt-hls.test.ts src/lib/online-subtitles.test.ts`：**20 pass / 0 fail**；
- `bun test src/lib`：1013 pass / 35 skip / 1 fail —— 失败项为
  `ffmpeg-decode.test.ts` 的 `subtitle tracks do not fail audio/video decode validation`
  （本机 `bin/ffmpeg.exe` 不再对 `-map 0` 报 `Encoder not found`，与本次改动无关，改动前即失败）。

## 已知未覆盖

- 腾讯字幕目前没有真机会话验证（匿名 getvinfo `sfl.cnt=0`）；拼接逻辑由离线用例覆盖
  （多镜像、相对/绝对 cue、分片 404）。
- 只处理 WebVTT；腾讯软字幕其它格式不存在，暂不需要 SRT→VTT 反解。

## 回滚

回退 `v0.2.85`（腾讯字幕行重新被过滤，不下载）。
