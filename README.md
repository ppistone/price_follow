# 价格 / 库存跟踪看板（price_follow）

一个左右框架的轻量看板：左侧是「标的」菜单（可持续添加），右侧显示对应内容（折线图 + 数据明细 + 来源说明）。纯静态、零依赖，可直接用 GitHub Pages 托管。

## 目录结构

```
price_follow/
├── index.html                # 看板本体（数据驱动）
├── data/
│   └── al_inventory.csv       # 国内电解铝库存数据（date,value,source）
└── README.md
```

## 数据格式

每个标的对应一个 CSV，三列：

```
date,value,source
2026-09-10,76.80,Mysteel周度
2026-09-17,70.50,Mysteel周度
```

- `source` 字段决定图中不同折线 / 颜色（同一标的可叠加多条序列，如历史月度 + 最新周度）。
- 第一行为表头；日期用 `YYYY-MM-DD`。

## 如何添加一个新标的

1. 在 `data/` 下新建一个 CSV（如 `data/cu.csv`），按上面的格式填数据。
2. 打开 `index.html`，在 `<script>` 顶部的 `TARGETS` 数组里追加一项：

```js
{
  id: "cu",
  name: "沪铜主力（元/吨）",
  unit: "元/吨",
  csv: "data/cu.csv",
  sources: ["上期所"],          // 与 CSV 里的 source 对应，决定颜色顺序
  note: "数据说明…",
  fallback: [ ["2026-01-02",70000,"上期所"] ]  // 本地直接打开时的兜底数据
}
```

左侧菜单会自动出现新标的，无需改其他代码。

## 电解铝库存数据来源说明

- **历史月度（2023-01 ~ 2026-05）**：SMM / 国联民生证券研报整理（三个皮匠报告），为券商二次整理数据，作历史参考。
- **最新周度（2026-09 起）**：Mysteel 铝专题快讯实采。Mysteel 免费页面仅保留最近数周，无法一次取到三年全量。
- 更新周度：用 `aluminum-social-inventory` skill 抓取 Mysteel，将新行 `日期,库存,Mysteel周度` 追加进 `data/al_inventory.csv` 即可。

## 本地预览

推荐起一个本地服务器（避免 `file://` 下 fetch 受限）：

```bash
cd price_follow
python -m http.server 8000
# 浏览器打开 http://localhost:8000
```

也可以直接双击 `index.html` 打开——此时会用 `TARGETS` 里的 `fallback` 内联数据兜底。

## 部署 / 推送

本仓库推送到 `https://github.com/ppistone/price_follow`。启用 GitHub Pages：

> 仓库 Settings → Pages → Source 选 `main` 分支根目录 → Save

之后访问 `https://ppistone.github.io/price_follow/`。
