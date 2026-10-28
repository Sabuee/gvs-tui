# TUI v0.2.88

配套网关 **v3.9.120**（最低 v3.9.119）。修 **画质表全标「软字幕」**。

## 问题

网关 `formats[]` 里每行都带 `caption_probe`，它表示「这行是用哪种 persona 探的」——
探测默认就是 `soft`，所以**几乎所有行都是 `caption_probe=soft`**。而 TUI 的
`qualityCaptionText` 在拿不到已证实的 `caption` 时会回退到 probe，于是整张画质表
全被标成「软字幕」，连优酷这种只有内嵌硬字幕的片源也躲不过。

真正能证明字幕类型的是 `caption`（网关按实际下发的 rendition + `sfl` 证据才写）。

## 改动

- `qualityCaptionText(caption, captionProbe, probeMixed)`：只认 `caption`（`soft`/`hard`）；
  仅当同一份目录里**同时存在软/硬两种探测档**（`probeMixed`）时才用 probe 区分显示，
  单一探测档一律不显示字幕列；
- `qualitiesFromTencentFormats`：构建完 rows 后统计 probe 集合，>1 时给每行打
  `captionProbeMixed=true`；`Quality` 类型同步加该字段；
- `qualityChoiceLabel` 改用 `qualityCaptionText(q.caption, q.captionProbe, q.captionProbeMixed)`；
- 测试：`tencent-quality-selection.test.ts` 的「未证实 probe」用例改为断言
  `qualityCaptionText('', 'soft') === ''`、`qualityChoiceLabel === 'FHD'`；
  软/硬两档并存的用例保持「软字幕 / 硬字幕」区分。

## 行为对照

| 场景 | 之前 | 现在 |
|---|---|---|
| 目录只有 soft 探测档（默认） | 全部标「软字幕」 | 不显示字幕列 |
| 同一目录软 + 硬两档并存 | 分别显示软/硬 | 保持不变 |
| 网关证实 `caption=soft`（有 sfl 真轨） | 「软字幕」 | 「软字幕」 |
| 优酷硬字幕片源（`subtitle_lang=default`） | 误标软字幕 | 不显示 |

## 验证

- `bunx tsc --noEmit` 无错误；
- `bun test src/lib`：**1025 pass / 35 skip / 0 fail**；
- 实网旁证（网关侧）：兰香如故 ep37 有 `sfl` 真轨 → 只有实际下发那一行 `caption=soft`；
  嫁金钗 / 哪吒 / 甄嬛传等硬字幕片源一个 `caption` 都没有。

## 回滚

回退 `v0.2.87`。
