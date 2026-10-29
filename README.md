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
2024-08-12,81.4,Mysteel周度·历史(2022-2025)
2026-09-10,76.8,Mysteel周度·实采(2026)
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

看板内「国内电解铝库存（万吨）」含两条真实序列（同一张图，不同颜色），**全部为 Mysteel 官方「电解铝社会库存」口径（万吨），真实值、非读图估算**：

- **Mysteel周度·历史(2022-2025)**：取自国盛证券周报图表（资料来源 Mysteel）的按周取样序列（周序 1/4/7/…/49），经多个独立 Mysteel 周度实采点交叉验证（2024-01-27=41.2、2024-08-12=81.4、2023-02-16=121.2 万吨均吻合）。日期按「每年 1月1日=第1周，日期=1月1日+(周序-1)×7天」近似标注（每周四发布）。
- **Mysteel周度·实采(2026)**：Mysteel 官网「电解铝周度数据」快讯原文实采值（3-20=152.9、3-27=137.1、6-15=129.8、8-20=84.9、8-27=82.1、9-03=78.9、9-10=76.8、9-17=70.5、9-24=65.1、10-15=58.6 万吨）。

**缺口说明**：2026 年 1-2 月、4-5 月、7 月暂无公开官方实采值，折线为分段真实点、未做插值。Mysteel 免费页面仅保留最近数周实采，后续可用 `aluminum-social-inventory` skill 每周补实采点，追加进 `data/al_inventory.csv` 即可（注意同时同步 `index.html` 里 `fallback` 内联兜底）。

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
