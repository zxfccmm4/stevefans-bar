# SteveFans · X 粉丝点阵看板

一个 [Metrix.Bar](https://metrix.bar) 菜单栏看板：圆角卡片、5×7 点阵粉丝数、数字滚动动画、X 认证徽章和 14 天面积趋势图。

`.bar` v0 单文件看板，无构建步骤、无后端、无鉴权。

![protocol](https://img.shields.io/badge/protocol-v0-4a9eff)
![license](https://img.shields.io/badge/license-MIT-black)

---

## 预览

截图使用示例数据；本机趋势需首次打开看板后积累两天以上采样才会显示面积图。

| 浅色 | 深色 |
| :---: | :---: |
| ![浅色主题](./screenshots/stevefans-light.png) | ![深色主题](./screenshots/stevefans-dark.png) |

---

## 功能

- **点阵数据层**：粉丝数、FOLLOWERS / FOLLOWING / POSTS / JOINED、涨跌和错误标签都用 5×7 点阵绘制（粉丝数最大 5 DIP 方格 + 2 DIP 缝隙，标签 1 DIP 点，涨跌 1.25 DIP 点，统计值 2 DIP 点）；昵称、`@handle` 和中文提示保留原生字体。
- **数字滚动动画**：粉丝数首次加载和数值变化时用 650ms 缓出滚动到新值；变化时同步刷新涨跌。
- **X 认证徽章**：认证账号使用官方 X 蓝色认证徽章，而不是普通对勾。
- **Lobe 卡片**：Block 式填充表面、12 DIP 圆角、柔和阴影和 1px 细边框；第一张卡片带淡 accent 底色；两张卡片可放进 560 DIP 面板，账号多时滚动。
- **多账号**：`users` 输入逗号/空格分隔，每个账号一张卡片；第一个账号决定菜单栏显示的数字。
- **账号信息**：X 头像、昵称、`@handle`、认证标识、FOLLOWING / POSTS / JOINED。
- **趋势图**：14 天原生面积图始终显示；采样不足两天时显示空图表和“从今天开始采样”，有数据后自动切换为面积图，并显示 7 天涨跌。
- **深浅色自适应**：使用宿主主题语义 token，卡片边框按外观使用 `#00000014` / `#ffffff1a` 的 1px alpha 描边。
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
- 第一张卡片使用淡 accent 底色 + 1px accent 边框，其余卡片使用 surface 底色和细 alpha 描边。
- Lobe 细节：24 DIP 无边框圆形图标按钮、Tag 式涨跌胶囊、带居中标题的 1 DIP Divider；动效按 Lobe Motion token（统计值 200ms、卡片 300ms、粉丝数 650ms）。
- 卡片结构：头像 + 昵称/`@handle` + 认证徽章 + 7 天涨跌；点阵粉丝数；点阵 FOLLOWING / POSTS / JOINED；14 天面积图。
- 点阵数字宽度自适应：4 位 159×47、5 位 200×47、6 位 243×47（DIP），最大缩放 7。
- 标签点阵 2× 缩放（1 DIP 点），涨跌 2.25× 缩放（1.25 DIP 点），统计值 3× 缩放（2 DIP 点），粉丝数 7× 缩放（5 DIP 点 + 2 DIP 缝隙）。
- 粉丝数滚动动画 650ms，使用 ease-out 曲线；卡片、涨跌和统计值增加 200–300ms 过渡，首次加载从约 88% 的目标值开始，避免大数字直接跳变。
- 趋势图高度 44 DIP，不显示坐标轴和网格，只保留 accent 面积与 1.5 DIP 线；没有数据时显示同高的空图表框。

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

A [Metrix.Bar](https://metrix.bar) dashboard with rounded Lobe cards, a 5×7 dot-matrix follower count, rolling-number animation, X verification badge and a 14-day area sparkline.

Screenshots above use sample data; the local 14-day chart appears once two daily samples have accumulated.

- **Dot-matrix data** — the follower count, labels, stats, delta and error tags are drawn with a 5×7 font; names, handles and Chinese hints stay native.
- **Rolling numbers** — the follower count animates over 650ms on first load and whenever it changes.
- **X verification** — verified accounts use the official X blue badge instead of a plain check.
- **Lobe cards** — Block-style filled surface, 12 DIP radius, soft shadow and 1px border; the first card has a pale accent tint; two cards fit a 560 DIP panel.
- **Multi-account** — comma/space-separated handles in the `users` input; the first one drives the menu bar number.
- **Trend chart** — a 14-day native area chart is always present; before two samples it shows an empty chart, then switches to the chart with a 7-day delta.
- **Theme aware** — semantic theme tokens plus `#00000014` / `#ffffff1a` 1px card borders, following the system appearance.
- **Refresh** — every 10 minutes, plus a catch-up refresh when the panel opens after 2 minutes; `Cmd+R` for manual refresh.

Data comes from the public `https://api.fxtwitter.com/<handle>` endpoint; avatars from `pbs.twimg.com`. No keys or tokens are embedded. Install by opening `stevefans.bar` with Metrix.Bar, or POST it to `/v0/dashboards` as shown above.
