# Desktop v0.1.91

配套 TUI **v0.2.88** / 网关 **v3.9.120**（最低 v3.9.119）。桌面端与 TUI 共用 `src/lib`，
这一版修掉 **画质表全标「软字幕」** 的显示问题。

## 问题

网关 `formats[]` 每行都带 `caption_probe`（表示用哪种 persona 探的），探测默认是 `soft`，
所以几乎每行都是 `caption_probe=soft`。旧逻辑在拿不到已证实的 `caption` 时回退到 probe，
于是整张画质表全标「软字幕」—— 连优酷这种只有内嵌硬字幕的片源也显示软字幕。

## 改动（共用库）

- `qualityCaptionText` 只认已证实的 `caption`；仅当同一目录里软/硬两档并存
  （`captionProbeMixed`）时才用 probe 区分；
- `qualitiesFromTencentFormats` 统计 probe 集合，>1 时才置 `captionProbeMixed`；
- `Quality` 类型补 `captionProbeMixed`。

## 桌面端影响

- 画质列表、画质选择摘要（`qualityChoiceLabel`）不再对硬字幕/未证实片源显示「软字幕」；
- 腾讯软字幕勾选区（`Quality.vue`）不受影响：仍按 `subtitles[]` 里**真实可下载的轨**显示，
  没有轨时显示「当前未返回可下载的软字幕轨」；
- 优酷不涉及字幕展示（网关 `has_soft_subtitle` 实测恒为 false）。

## 验证

- `bunx tsc --noEmit` 无错误；
- `bun test src/lib`：1025 pass / 35 skip / 0 fail；
- 安装包由桌面端 Release 工作流构建（`desktop-v*` 标签与 `app/package.json` 版本一致）。

## 回滚

回退 `desktop-v0.1.90`。
