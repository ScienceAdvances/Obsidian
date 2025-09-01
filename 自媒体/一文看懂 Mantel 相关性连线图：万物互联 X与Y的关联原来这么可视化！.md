
## Mantel 检验介绍

Mantel 检验是一种用于评估两个矩阵之间相关性的统计方法，主要应用于生态学研究中。它的核心思想是：
1. 计算两个矩阵各自的距离矩阵（或相似性矩阵）
2. 检验这两个距离矩阵之间是否存在显著的相关性
3. 通过置换检验（permutation test）来评估相关性的显著性

![image.png](https://s2.loli.net/2025/08/28/gqsWBkYnFdNxXRf.png)

## 安装R包

如果安装不上可以找我要本地安装包
```js
BiocManager::install(c("Hy4m/linkET","tidyverse"))
```

## 生成用于绘图的随机数据

生成两个矩阵x是3个基因为列名30行，y为6个特征（Env 可以是环境因子，免疫浸润等等内容，总之是30个样本的另一种特征）

```js
# 设置随机种子，确保结果可重复
set.seed(123)

# 生成样本数量
n_samples <- 30

# 生成x矩阵 (3个基因，30个样本)，基因名改为Gene1、Gene2、Gene3
x <- data.frame(
  Gene1 = rnorm(n_samples, mean = 5, sd = 1.2),
  Gene2 = rnorm(n_samples, mean = 3, sd = 0.8),
  Gene3 = rnorm(n_samples, mean = 7, sd = 1.5)
)


# 生成y矩阵 (增加到6个环境因子)
# 使y与x存在一定相关性，增加数据的生物学意义
y <- data.frame(
  Env1 = x$Gene1 * 0.3 + x$Gene2 * 0.2 + rnorm(n_samples, 0, 0.5),
  Env2 = x$Gene2 * 0.4 + x$Gene3 * 0.1 + rnorm(n_samples, 0, 0.5),
  Env3 = x$Gene1 * 0.2 + x$Gene3 * 0.3 + rnorm(n_samples, 0, 0.5),
  Env4 = x$Gene1 * 0.1 + x$Gene2 * 0.1 + x$Gene3 * 0.2 + rnorm(n_samples, 0, 0.5),
  Env5 = x$Gene1 * 0.25 + rnorm(n_samples, 0, 0.6),
  Env6 = x$Gene3 * 0.35 + rnorm(n_samples, 0, 0.5)
)

# 转换为矩阵格式
x <- as.matrix(x)
y <- as.matrix(y)
```


## 进行Mantel 检验
spec_select含义，把x矩阵的列分组后再与y进行Mantel 检验，为什么要分组，根据研究内容不同，可能某几列（某几个基因）是一个通路一起发挥作用。

同理env_select也可以分组

原理如下，因此选择不同的列进行分组，得到的x或者y距离矩阵是不同的，从而检验结果也是不同的。

Mantel 检验核心是解决 “两组多变量数据的整体差异模式是否相关” 的问题：首先将分别代表 “特征”（如基因表达、物种丰度）和 “环境 / 表型” 的原始矩阵，转换为对应相同样本的距离矩阵（描述样本间在特征或环境上的差异）；接着计算两个距离矩阵的相关性系数（Mantel's r，反映关联强度）；最后通过多次随机置换其中一个矩阵、生成随机 r 值分布，判断实际 r 值的统计学显著性（P 值），最终明确两组变量的整体差异模式是否存在显著关联，广泛用于生态学、遗传学等领域（如基因表达模式与环境因子的关联分析）。



```js
mantel <- linkET::mantel_test(
    spec = x, env = y,
    spec_select = list(Gene1 = 1, Gene2 = 2, Gene3 = 3)
) %>%
    dplyr::mutate(rd = cut(r, breaks = c(-Inf, 0.3, 0.6, Inf), labels = c("< 0.3", "0.3 - 0.6", ">= 0.6")),
     pd = cut(p, breaks = c(-Inf, 0.01, 0.05, Inf), labels = c("< 0.01", "0.01 - 0.05", ">= 0.05"))
    )
```


## 美图

```js
# 使用qcorrplot函数初始化相关性绘图
# correlate(y)：计算环境因子矩阵y中各变量的Pearson相关系数
# type = "lower"：只显示相关性矩阵的下三角部分（避免重复显示上三角）
# diag = TRUE：显示对角线（变量与自身的相关性，值为1）

linkET::qcorrplot(correlate(y), type = "lower", diag = TRUE) +
  # 添加方形热图元素，用颜色深浅表示相关系数大小
  geom_square() +
  # 添加曲线连接线，展示Mantel检验结果
  # aes(color = pd, size = rd)：用颜色区分p值分组，用线条粗细区分r值分组
  # data = mantel：使用Mantel检验的结果数据
  # curvature = nice_curvature()：自动调整曲线曲率，使连线更美观
  geom_couple(aes(color = pd, size = rd), data = mantel, curvature = nice_curvature()) +
  # 添加显著性标记（星号）
  # sep = "\n"：不同显著性水平的标记分行显示
  # size = 3：标记文字大小
  # sig_level = c(0.05, 0.01, 0.001)：显著性水平阈值，对应*, **, ***
  # sig_thres = 0.05：只显示p < 0.05的显著关联
  # color = "black"：标记颜色为黑色

  geom_mark(sep = "\n", size = 3, sig_level = c(0.05, 0.01, 0.001), sig_thres = 0.05, color = "black") +

  # 设置热图颜色渐变，使用RdBu色板（红-白-蓝），红色表示正相关，蓝色表示负相关
  scale_fill_gradientn(colours = RColorBrewer::brewer.pal(11, "RdBu")) +
  # 手动设置线条粗细对应的Mantel's r分组
  # values = c(0.5, 1, 2)：分别对应"< 0.3"、"0.3 - 0.6"、">= 0.6"三个组
  scale_size_manual(values = c(0.5, 1, 2)) +
  # 手动设置线条颜色对应的Mantel's p分组
  # color_pal(3)：生成3种区分明显的颜色，分别对应"< 0.01"、"0.01 - 0.05"、">= 0.05"
  scale_colour_manual(values = color_pal(3)) +
  # 调整图例样式和顺序
  # size参数：设置Mantel's r的图例，标题为"Mantel's r"，图例符号颜色固定为深灰
  # color参数：设置Mantel's p的图例，标题为"Mantel's p"，图例符号大小固定为3
  # fill参数：设置Pearson相关系数的颜色条图例，标题为"Pearson's r"
  # order：指定图例显示顺序（1为Mantel's p，2为Mantel's r，3为Pearson's r）

  guides(size = guide_legend(title = "Mantel's r", override.aes = list(color = "grey35"), order = 2), color = guide_legend(title = "Mantel's p", override.aes = list(size = 3), order = 1), fill = guide_colorbar(title = "Pearson's r", order = 3))
```

![image.png](https://s2.loli.net/2025/08/28/EYMga6Fi1y4QecC.png)
## Reference
```js
https://github.com/DongPingXiJin/Mantel-test
https://doi.org/10.1111/2041-210x.12018
```