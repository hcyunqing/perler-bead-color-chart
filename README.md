# perler-bead-color-chart

perler-bead-color-chart 收录了 MARD、COCO、漫漫、盼盼、咪小窝五个主流拼豆（Perler Beads / 熨烫豆）品牌的共 **291 个颜色**的跨品牌色号对照表，每个颜色附标准 HEX 色值，并提供 CSV 与 JSON 两种机器可读格式。

拼豆创作中最常见的痛点是：图纸标注的是 A 品牌色号，手里买的却是 B 品牌的豆子，而各品牌之间色号完全不能互换。本数据集按「同一实测颜色」将五个品牌逐色对齐，可以：

- 按手头品牌的色号，查出图纸所需颜色在本品牌中的对应色号
- 为程序/脚本提供色号查询与转换数据（JSON 直接可用）
- 作为拼豆工具类应用、图纸生成器的内置数据源

数据由在线拼豆图纸平台 **豆画**（[https://idouhua.cn](https://idouhua.cn)）整理维护，免费开放，采用 [CC BY 4.0](./LICENSE) 授权——可自由使用、转载与二次分发，请保留来源署名。

## 数据文件

| 文件 | 格式 | 说明 |
|---|---|---|
| [`data/color-chart.csv`](./data/color-chart.csv) | CSV | 主对照表：291 行 × 6 列（hex + 5 品牌色号），按 HEX 升序 |
| [`data/color-chart.json`](./data/color-chart.json) | JSON | 主对照表数组版，字段同上 |
| [`data/brands/mard.json`](./data/brands/mard.json) | JSON | MARD 单品牌：291 条 `code ↔ hex` |
| [`data/brands/coco.json`](./data/brands/coco.json) | JSON | COCO 单品牌 |
| [`data/brands/manman.json`](./data/brands/manman.json) | JSON | 漫漫 单品牌 |
| [`data/brands/panpan.json`](./data/brands/panpan.json) | JSON | 盼盼 单品牌 |
| [`data/brands/mixiaowo.json`](./data/brands/mixiaowo.json) | JSON | 咪小窝 单品牌 |

## 字段说明

| 字段 | 类型 | 含义 |
|---|---|---|
| `hex` | string | 该颜色的 HEX 色值（如 `#01ACEB`），跨品牌对齐的基准键 |
| `mard` | string | MARD 品牌色号（如 `C5`） |
| `coco` | string | COCO 品牌色号（如 `H07`） |
| `manman` | string | 漫漫 品牌色号（如 `D7`） |
| `panpan` | string | 盼盼 品牌色号（如 `34`） |
| `mixiaowo` | string | 咪小窝 品牌色号（如 `34`） |

已知数据备注：漫漫（manman）存在 1 个色号对应两种实测颜色的情况（`S4` → `#7FCD9D` / `#F3C1C0`），为官方色号体系本身的重号，数据中如实保留两条记录。

## 使用示例

Python（pandas 读取 + 跨品牌换算）：

```python
import pandas as pd

df = pd.read_csv("data/color-chart.csv")

# 手里有 MARD C5，想知道 COCO 对应什么色号？
code = df.loc[df["mard"] == "C5", ["hex", "coco"]]
print(code)  # hex=#01ACEB, coco=H07

# 图纸要求 COCO H07，用手头 MARD 豆子怎么替换？
code = df.loc[df["coco"] == "H07", "mard"]
print(code)  # C8
```

JavaScript / TypeScript：

```ts
import chart from "./data/color-chart.json";

const row = chart.find((r) => r.mard === "C5");
console.log(row?.coco); // "H07"
```

在线查询与换算：可直接使用豆画色号库 <https://idouhua.cn/colors>，支持按色号搜索、色卡预览与图纸内一键替换。

## 跨品牌换算方法

1. **以 HEX 为基准**：本表已按实测颜色对齐，直接查表即可，无需自行计算。
2. **无法对齐时**（例如你只想要近似的替代色）：使用 RGB 最近色匹配——将目标色号转为 RGB，与数据集中所有颜色算欧氏距离（建议加权：`ΔR²×0.3 + ΔG²×0.59 + ΔB²×0.11`，人眼对绿色更敏感），取最小值对应行。
3. **色系规律**：MARD 以字母分区（A=白灰系、B=黑灰系、C=蓝系…），COCO 以 H/G/B 等字母分区，品牌间不存在线性映射，不要按序号直接换算。

更详细的换算讨论见 [docs/convert.md](./docs/convert.md)。

## 数据来源与维护

- 数据由豆画团队从各品牌官方色卡整理校对，随官方色卡更新而更新。
- 发现错误或缺失？欢迎提 Issue 或 PR，或在 [豆画](https://idouhua.cn) 联系我们。
- 如果本数据集对你有帮助，欢迎点一个 Star，也欢迎在作品/文章中引用。

## License

数据内容采用 [CC BY 4.0](./LICENSE)（署名 4.0 国际）授权：允许任何用途（含商用），需注明来源「豆画 idouhua.cn」。
