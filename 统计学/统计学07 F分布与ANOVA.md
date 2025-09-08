## F分布

![](https://cdn.nlark.com/yuque/__latex/3c2fdc3c5133a10a6386c7585e422729.svg)

其中，![](https://cdn.nlark.com/yuque/__latex/dc5b86e9a9500d022d93077952cef8e1.svg)服从自由度为![](https://cdn.nlark.com/yuque/__latex/8a4fe6a305bd495b10464ebb77c04cc6.svg)的卡方分布，![](https://cdn.nlark.com/yuque/__latex/eb507217b0efd61b0414242624ec8158.svg)服从自由度为![](https://cdn.nlark.com/yuque/__latex/081cc6fa3cdf1d79f9134242293306a2.svg)的卡方分布

F分布重要用来做Anova，即用方差的思想去做多总体均值的化较 Anova : analysis of variance

方差分析（ANOVA）又称“变异数分析”或“F检验”

## F值的求法

1. 把 n 组数据放在一起,看成一个总体,算出这个总体的均值![](https://cdn.nlark.com/yuque/__latex/d5c5b0ffc1a819c48e80616ba2b41e83.svg)
2. 计算出每组数据的组内平均值![](https://cdn.nlark.com/yuque/__latex/780565603d0522bdc1975a0c13d8e26f.svg)
3. 计算出组间差异

![](https://cdn.nlark.com/yuque/__latex/b8a5cb3a31b8930a3b5fba27c8822548.svg)

4.计算出组内差异

![](https://cdn.nlark.com/yuque/__latex/665c207e57ca01477a23137b20d02a93.svg) 为每组的平均值

![](https://cdn.nlark.com/yuque/__latex/843cee63612fcca0ccd9796dff1cb806.svg)

5.计算F值

![](https://cdn.nlark.com/yuque/__latex/c4d569a40149340564a56d2d4df92f39.svg)

其中n为组数，m为全部的观测值数

## 手动ANOVA

设有三台机器，用来生产规格相同的铝合金薄板，取样，测量薄板的厚度。得结果如表。

|   |   |   |
|---|---|---|
|机器A|机器B|机器C|
|0.236|0.257|0.258|
|0.238|0.253|0.264|
|0.248|0.255|0.259|
|0.245|0.254|0.267|
|0.243|0.261|0.262|

检验假设 (![](https://cdn.nlark.com/yuque/__latex/b221a01585cbaaee5f26a1afae4ffff9.svg));

![](https://cdn.nlark.com/yuque/__latex/99d1ee51cd4f3fcc4f17fa41fd1630a4.svg)

![](https://cdn.nlark.com/yuque/__latex/ff6beb4810cc0335440b63cb3359541c.svg)

![](https://cdn.nlark.com/yuque/__latex/e05fdef7efb7421317e73b036aaf1736.svg)

![](https://cdn.nlark.com/yuque/__latex/731ccaa69bd036af05aafc924fc263a2.svg)

## Julia代码实现

```
using DataFrames, CSV,HypothesisTests,Distributions,CairoMakie
txt="""
机器A,机器B,机器C
0.236,0.257,0.258
0.238,0.253,0.264
0.248,0.255,0.259
0.245,0.254,0.267
0.243,0.261,0.262
"""
data = CSV.read(IOBuffer(txt),DataFrame)
u0 = mean(Matrix(data))
ssb(data::AbstractDataFrame)=mapcols(col->length(col)*(mean(col)-u0)^2,data) |> Matrix |> sum
ssw(data::AbstractDataFrame) = mapcols(col -> sum((col .- mean(col)).^2),data) |> Matrix |> sum
n=ncol(data)
m=size(data)[1] * size(data)[2]
F=(ssb(data)/(n-1)) / (ssw(data)/(m-n))
# 32.916666666666714
```

## F分布图

```
let
    f=Figure()
    Axis(f[1, 1])
    x = 0:0.01:35
    lines!(x, pdf.(FDist(2,12), x), label = "df1 = 2; df1 = 12")
    vlines!(32.92,ymax=0.05,color=:red)
    axislegend()
    current_figure()
end
```

![image.png](https://s2.loli.net/2025/09/08/7PSQkFdKnZzCgcD.png)


## 使用函数做ANOVA

Perform one-way analysis of variance test of the hypothesis that that the `groups` means are equal.

### Julia

```
HypothesisTests.OneWayANOVATest(eachcol(data)...)
# One-way analysis of variance (ANOVA) test
# -----------------------------------------
# Population details:
#     parameter of interest:   Means
#     value under h_0:         "all equal"
#     point estimate:          NaN

# Test summary:
#     outcome with 95% confidence: reject h_0
#     p-value:                     <1e-04

# Details:
#     number of observations: [5, 5, 5]
#     F statistic:            32.9167
#     degrees of freedom:     (2, 12)
```

Python

```
import pandas as pd
from io import StringIO
from statsmodels.formula.api import ols                    # 最小二乘法拟合
from statsmodels.stats.anova import anova_lm               # 方差分析
from statsmodels.stats.multicomp import pairwise_tukeyhsd  # post Hoc t_test

txt="""
机器A,机器B,机器C
0.236,0.257,0.258
0.238,0.253,0.264
0.248,0.255,0.259
0.245,0.254,0.267
0.243,0.261,0.262
"""
df=pd.read_csv(StringIO(txt))
df_melt=df.melt(id_vars=None, value_vars=df.columns, ignore_index=True)

model = ols('value ~ C(variable)', data = df_melt).fit()  # 最小二乘法拟合
print(model.params)
# Intercept             0.242
# C(variable)[T.机器B]    0.014
# C(variable)[T.机器C]    0.020
# dtype: float64
anova_table = anova_lm(model, type = 2)                 # 方差分析
pd.DataFrame(anova_table)                               # 查看方差分析结果
#                df    sum_sq   mean_sq          F    PR(>F)
# C(variable)   2.0  0.001053  0.000527  32.916667  0.000013
# Residual     12.0  0.000192  0.000016        NaN       NaN
pt=pairwise_tukeyhsd(df_melt['value'], df_melt['variable'])
print(pt)
# Multiple Comparison of Means - Tukey HSD, FWER=0.05
# ===================================================
# group1 group2 meandiff p-adj   lower  upper  reject
# ---------------------------------------------------
#    机器A    机器B    0.014 0.0004  0.0073 0.0207   True
#    机器A    机器C     0.02    0.0  0.0133 0.0267   True
#    机器B    机器C    0.006 0.0836 -0.0007 0.0127  False
# ---------------------------------------------------
```