# 拼豆跨品牌色号换算方法

> 本文是 [perler-bead-color-chart](../README.md) 数据集的配套说明，回答「不同品牌的拼豆色号怎么换算」这一问题。

## 一、为什么色号不能直接互换

各品牌色号体系互相独立：

- **MARD**：字母（色系）+ 数字（序号），如 `A01`（白灰系）、`C5`（蓝系）、`H7`（黑灰系）
- **COCO**：字母分区 + 数字，如 `H07`、`G08`、`B09`
- **漫漫 / 盼盼 / 咪小窝**：纯数字序号体系

同样叫「C5」的色号，在不同品牌是不同的颜色；同样颜色的豆子，在不同品牌编号完全不同。**不存在任何线性映射关系**，唯一可靠的方法是按实测颜色对齐。

## 二、查表法（推荐，100% 准确）

本数据集已经把 291 个颜色按实测 HEX 逐色对齐，直接查表：

```
目标：图纸要求 COCO H07，手里只有 MARD

1. 在 color-chart.csv 中找到 coco = H07 的行
2. 该行 mard 列的值就是 MARD 侧对应色号（C8）
3. hex 列即该颜色的实测色值（#0F54C0），可用于核对
```

## 三、最近色匹配法（数据未覆盖时）

如果需要换算的颜色不在表内（新色号、停产色等），用 RGB 最近色匹配：

### 欧氏距离（基础）

```
dist = sqrt(ΔR² + ΔG² + ΔB²)
```

### 加权距离（推荐，更符合人眼感知）

人眼对绿色通道最敏感，红色次之，蓝色最弱：

```
dist² = ΔR² × 0.30 + ΔG² × 0.59 + ΔB² × 0.11
```

示例代码（Python）：

```python
import pandas as pd

df = pd.read_csv("data/color-chart.csv")

def hex_to_rgb(h):
    h = h.lstrip("#")
    return tuple(int(h[i:i+2], 16) for i in (0, 2, 4))

def nearest(target_hex, brand="mard"):
    tr, tg, tb = hex_to_rgb(target_hex)
    best, best_d = None, 1e9
    for _, row in df.iterrows():
        r, g, b = hex_to_rgb(row["hex"])
        d = 0.30*(r-tr)**2 + 0.59*(g-tg)**2 + 0.11*(b-tb)**2
        if d < best_d:
            best_d, best = d, row[brand]
    return best

print(nearest("#3FA7D6"))  # 返回 MARD 中最接近的色号
```

### 注意事项

- **在固定光照下取色**：豆子实测颜色受环境光影响，建议自然光下白平衡拍摄取色
- **同色系内匹配**：跨色系最近色（如把粉色匹配到红色系）观感会明显偏离，可先限定色系再匹配
- **少量误差可接受**：拼豆成品本身由离散色块构成，±1 色阶的差异在 5mm 豆幅面下通常不可见

## 四、图纸内一键替换

不需要手动换算：在豆画编辑器（idouhua.cn/canvas）中打开任意图纸，使用「色号替换」功能选择目标品牌，系统自动按最近色把全部色号转到该品牌，并重新统计所需豆子清单。
