![](https://cdn.nlark.com/yuque/0/2024/png/975582/1725008354870-6b5ee8b0-ef66-47c4-ba9b-cf65012514e7.png)

col：变量（如基因）row：样品

```
X：数据框。行是个体，列是数字变量
scale.unit：一个逻辑值。如果为 TRUE，则在分析之前将数据缩放为单位方差。这种相同规模的标准化避免了一些变量因其较大的测量单位而成为主导。它使变量具有可比性。
ncp：最终结果中保留的维数。
graph：一个逻辑值。如果为 TRUE，则显示图形。
```

```
get_eigenvalue(res.pca)：提取主成分的特征值/方差（即信息）的比例

fviz_eig(res.pca)：可视化特征值。

get_pca_ind(res.pca)，get_pca_var(res.pca)：分别提取个体和变量的结果。

fviz_contrib(res.pca): 提取对主成分贡献较多的观测。

fviz_pca_ind(res.pca)，fviz_pca_var(res.pca)：分别可视化结果个体和变量。

fviz_pca_biplot(res.pca)：制作主成分分析散点图biplot图。
```

# 每个样品点图

```
p_load(FactoMineR,factoextra)
iris.pca = PCA(iris[,-5], scale.unit = TRUE, ncp = 2, graph = F)
fviz_pca_ind(X = iris.pca, ## pca对象
  axes = 1:2, ## 展示的两个主成分
  geom = 'point', ## 展示individual的形式
  habillage = iris$Species, #分组 factor
  legend.title = 'Groups', ## 分组变量的title
  palette = 'lancet', ## 颜色面板
  addEllipses = T,  ## 是否绘制椭圆
  ellipse.level = 0.95, ## 椭圆的大小
  title = 'PCA Plot', ## 标题
  mean.point = T ## 不删除每个组的重心
  )+
  theme(plot.title = element_text(hjust = 0.5,size=20))
```

|   |   |
|---|---|
|参数|解释|
|X|PCA对象，可以来自多个包，如FactoMomeR; prcomp; princomp等|
|axes|一个长度为2的向量，指定绘制哪两个主成分|
|geom|指定图形要使用的几何图形的一种文本。允许的值是c("point"， "arrow"， "text")的组合。使用“point”(仅表示点);“text”只显示标签;c("point"， "text")或c("arrow"， "text")表示箭头和文本。使用c("arrow"， "text"))只对变量的图形有意义。|
|geom.ind, geom.var|as geom but for individuals and variables, respectively. Default is geom.ind = c("point", "text), geom.var = c("arrow", "text").|
|repel|布尔值，是否使用ggrepel来避免过度绘制文本标签|
|habillage|可选的**因子变量**，用于按组为观察结果着色。默认值为none。|
|legend.title|分组变量的title|
|palette|颜色|
|addEllipses|是否加椭圆|
|ellipse.level|椭圆的大小|
|ellipse.type|“椭圆”的类型；如果你想要置信椭圆而不是集中椭圆，请使用 ellipse.type ="confidence" 。置信椭圆是对置信区域的描述，浓度椭圆是对点分布的描述。置信椭圆的算法复杂，背后有很多繁杂的数学原理。置信椭圆的长短半轴，分别表示二维位置坐标分量的标准差（如经度的 σλ 和纬度的 σφ）。一倍标准差（1σ）的概率值是 68.3%，二倍标准差（2σ）的概率值为 95.5%；三倍标准差（3σ）的概率值是 99.7%。|
|col.ind, col.var|color for individuals and variables, respectively. Can be a continuous variable or a factor variable. 其中，col.ind也可以起到类似于habillage的作用。|
|fill.ind, fill.var|same as col.ind and col.var but for the fill color；其中，fill.ind也可以起到类似于habillage的作用。|
|alpha.ind, alpha.var|controls the transparency of individual and variable colors, respectively.|
|gradient.cols|用于n色梯度的颜色向量|
|label|是否标记样本的label，可取all或none|
|title|图的标题|
|mean.point|是否删除每个组的重心|
## 3D PCA
![](https://cdn.nlark.com/yuque/0/2024/png/975582/1725008496861-8d9618fa-68d9-4725-8b0c-50163db25c66.png)

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1725008496744-289c0b1d-fbc4-47be-a961-96050010cd4b.png)

```
library(pca3d)
# ------------------制作数据------------------
dt <- iris[, -5]
rownames(dt) <- paste0("sample", 1:nrow(dt))
dt

# ------------------计算PCA------------------
pca <- princomp(dt) # 变量在列
gr <- factor(iris[, 5]) # 分组信息
pcs <- pca$sdev^2 / sum(pca$sdev^2) * 100 #计算主成分贡献

# ------------------3D PCA plot------------------
pca3d(pca,
    palette=c("#1B9E77", "#D95F02", "#7570B3"),
    group = gr, new = T,
    axe.titles = sprintf(c("PC1: %.2f %%", "PC2: %.2f %%", "PC3: %.2f %%"), pcs[1:3]),
    show.centroids = T,
    legend = "right",
    show.plane = FALSE,
    show.shapes = TRUE
)
snapshotPCA3d("pca3d.png") # macbook需要安装XQuartz

# ------------------2D PCA plot------------------
pdf(file="pca2d.pdf")
pca2d(pca,
    group = gr, new = T,
    axe.titles = sprintf(c("PC1: %.2f %%", "PC2: %.2f %%"), pcs[1:2]),
    show.centroids = F,
    legend = "right",
    show.group.labels = F,
    show.shapes = TRUE,
    show.ellipses = T,
    show.plane = T,
    ellipse.ci = 0.95
)
dev.off()
```

# References

[https://mp.weixin.qq.com/s/OBwzcXm9zm6EUpIlcOdF-Q](https://mp.weixin.qq.com/s/OBwzcXm9zm6EUpIlcOdF-Q)

[https://mp.weixin.qq.com/s/ifgIN_7t_Y1pE-1WeFrFuA](https://mp.weixin.qq.com/s/ifgIN_7t_Y1pE-1WeFrFuA)