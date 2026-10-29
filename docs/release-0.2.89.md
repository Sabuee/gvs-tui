# TUI v0.2.89

配套网关 **v3.9.120**（最低 v3.9.119）。修 **腾讯「返回的实际版本与所选版本不同」误判**。

## 问题

腾讯下载前会把「实际下发的 rendition」和「所选档位」做一次比对，对不上就停下报
「返回的实际版本与所选版本不同」。比对里带了字幕一项，而所选字幕取的是
`caption_probe` —— 它只表示「这行是用软字幕还是硬字幕 persona 探的」，探测默认就是
`soft`，并不是已证实的字幕。

于是：画质表里只有探测值的档位（最常见），选定后实际返回没有 `caption` 字段，
`'soft' !== ''` 就判成不同版本，下载直接停。所选档和实际档其实是同一条。

## 改动

- `tencentDownloadSelection`：取流请求仍按探测方式带 `caption`（避免网关换到另一档），
  但返回值里**只保留目录已证实的 `caption`**（`soft` / `hard`）；只有 probe 的档位不带字幕。
- `equivalentTencentRendition`：比对改用 `selected.captionProbe || input.caption` 折算出的
  请求字幕（`requestedCaption`），与实际行的 `captionProbe` / `caption` 比；分集格式 ID
  重绑仍按请求的探测字幕匹配，档位不放宽。
- 测试新增「probe-only 选择在交付侧没有 caption 时不算 different」。

## 行为对照

| 场景 | 之前 | 现在 |
|---|---|---|
| 档位只有 `caption_probe=soft`，实际返回无 `caption` | 判成不同版本，停下载 | 判为同一条，正常下载 |
| 目录已证实 `caption=soft`，实际返回 `caption=soft` | 同一条 | 同一条 |
| 目录证实软、实际返回硬（真的不符） | 判不同 | 仍判不同 |
| 分集重新绑定格式 ID | 按请求字幕匹配 | 不变 |

## 验证

- `bunx tsc --noEmit` 无错误；
- `bun test --timeout 60000 src/lib`：**1026 pass / 35 skip / 0 fail**（1061 用例 / 138 文件）；
- PR 自带用例：`tencent-quality-selection` / `tencent-rendition-resolution` /
  `actual-version` / `media-tencent-probe` 共 55 通过。

## 回滚

回退 `v0.2.88`。
