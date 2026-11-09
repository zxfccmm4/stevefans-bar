# SteveFans · X 粉丝像素看板

一个 [Metrix.Bar](https://metrix.bar) 菜单栏看板：用 5×7 点阵像素显示 X（Twitter）账号的粉丝数、关注数、推文数，以及本机记录的粉丝涨跌。

`.bar` v0 单文件看板，无构建步骤、无后端、无鉴权。

![protocol](https://img.shields.io/badge/protocol-v0-4a9eff)
![license](https://img.shields.io/badge/license-MIT-black)

---

## 功能

- **点阵数字**：粉丝数用 Canvas 逐格绘制（5×7 字模，8 DIP 方格 + 2 DIP 缝隙），浅色主题黑字、深色主题白字。
- **多账号**：`users` 输入逗号/空格分隔，每个账号一张卡片；第一个账号决定菜单栏显示的数字。
- **账号信息**：X 头像、昵称、`@handle`、认证标识、FOLLOWING / POSTS / JOINED。
- **本机趋势**：每天采样一次粉丝数并保存在 `localStorage`（保留 31 天）；有两天以上数据后显示 7 天涨跌和像素柱状图（14 天，涨 accent、跌 danger）。
- **深浅色自适应**：使用宿主主题语义 token（`text` / `textMuted` / `border` / `accent`），卡片边框按外观使用 `#00000010` / `#ffffff12` 的 alpha 描边。
- **刷新策略**：静态 10 分钟轮询；打开面板且数据超过 2 分钟时补刷；`Cmd+R` 手动刷新。
- **错误分层**：单个账号失败只影响该卡片，全部失败时显示问题页并把原因写入 `tray.error`。

## 安装

1. 从 Releases 或仓库下载 [`stevefans.bar`](./stevefans.bar)。
2. 用 Metrix.Bar 打开它（或在看板目录里双击）。

也可以用本地 API 安装：

```sh
export MB=http://127.0.0.1:6387
export TOKEN=...   # Metrix.Bar → Agent → Copy Prompt 里的 token

curl -fsS -X POST "$MB/v0/dashboards" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: text/plain" \
  --data-binary @stevefans.bar
```

## 配置

| 输入 | 类型 | 默认值 | 说明 |
| :--- | :--- | :--- | :--- |
| `users` | `string` | `st7evechou` | 一个或多个 X 用户名，逗号或空格分隔，不带 `@`。第一个账号决定菜单栏数字。 |

## 数据来源

- 账号数据：`https://api.fxtwitter.com/<handle>`（公开接口，无需鉴权）。
- 头像：`https://pbs.twimg.com`。
- 趋势：仅保存在本机 `localStorage`（键名 `fans:<handle>`），不上传、不跟随 `.bar` 导出；首次打开看板后才开始积累。

> 只有你填写的 handle 会发给 FxTwitter；看板不包含任何 API key 或 token。

## 权限

```xml
<permissions>
  <network origin="https://api.fxtwitter.com" />
  <network origin="https://pbs.twimg.com" />
</permissions>
```

`<script apis="localStorage shell match-media">`：本地趋势存储、打开 X 链接、跟随系统深浅色。

## 外观

- 面板跟随即时系统外观：浅色白底黑字，深色黑底白字。
- 第一个账号使用 `accent` 边框标识。
- 点阵数字宽度自适应：5 位 290×70、4 位 230×70、6 位 280×56（DIP）。

## 开发与校验

看板是单文件，修改后可直接用 Metrix.Bar 本地 API 校验：

```sh
# 语法/结构
curl -fsS -X POST "$MB/v0/validate" \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: text/plain" \
  --data-binary @stevefans.bar

# 隔离运行时 + UI 树（会执行一次真实网络请求）
jq -n --rawfile source stevefans.bar '{source:$source,tree:true,scheduled:true}' \
  | curl -fsS -X POST "$MB/v0/check" \
      -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
      --data-binary @- 
```

替换已安装版本时使用 `GET /v0/dashboards/{id}/source` 返回的 `ETag` 作为 `If-Match`。

## 兼容性

- 在 Metrix.Bar **0.1.2**（protocol 0，macOS）上开发并验证。
- 面板与侧边栏跨平台可用；菜单栏标题/图标为 macOS 行为，Windows/Linux 的托盘展示取决于系统。

## License

[MIT](./LICENSE)

---

## English

A single-file [Metrix.Bar](https://metrix.bar) dashboard that shows X (Twitter) follower, following and post counts as a 5×7 dot-matrix readout, with multi-account cards and a locally sampled 7-day trend.

- **Pixel readout** — the follower count is drawn cell by cell on a Canvas (8 DIP cells, 2 DIP gaps), black on light and white on dark.
- **Multi-account** — comma/space-separated handles in the `users` input; the first one drives the menu bar number.
- **Local trend** — one sample per day in `localStorage` (31 days); after two samples it renders a 7-day delta and pixel bars.
- **Theme aware** — semantic theme tokens plus `#00000010` / `#ffffff12` card borders, following the system appearance.
- **Refresh** — every 10 minutes, plus a catch-up refresh when the panel opens after 2 minutes; `Cmd+R` for manual refresh.

Data comes from the public `https://api.fxtwitter.com/<handle>` endpoint; avatars from `pbs.twimg.com`. No keys or tokens are embedded. Install by opening `stevefans.bar` with Metrix.Bar, or POST it to `/v0/dashboards` as shown above.
