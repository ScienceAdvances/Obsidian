```
# df50 50基因50样本 行名基因名，列样本名
sns.clustermap(df50,col_colors=['r']*25+['g']*25,cmap="vlag",col_cluster=False,z_score=1,xticklabels='')
```

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1725008460702-8050a907-5144-423c-9864-37dc5891ea67.png)

# z-score

数据标准化

data基因表达矩阵，行是基因，列是基因

scale()函数对列进行批量操作，目的是使gene表达值在所有样品间呈标准正态分布

参数

1.center默认TRUE，每一列减去其mean() 2.scale默认TRUE，每一列除以其sd()

```
t(data) %>% scale() %>% t()
```

Z-score 转换后，数值分布也相对均一了，每个基因在不同样品之间的表达的高低一目了然。但是不同基因之间就完全不可比了。

# ComplexHeatmap

```
hue <- c("#1AB3B3", "#FF6699", "#453B99", "#6B8F24", "#FF8C00")
top_annotation <- ComplexHeatmap::HeatmapAnnotation(
    Group = pdata$Group,
    col = list(Group = c(
        "HC" = hue[1],
        "ATB" = hue[2],
        "T6" = hue[3]
    )),
    gp = grid::gpar(col = NA)
)

plot_data = d[, rownames(pdata)] %>% t() %>% scale() %>% t()
range(plot_data)
color <- circlize::colorRamp2(
    breaks = c(-2, 0, 9), # 值范围断点
    colors = c("#1AB3B3", "white", "#FF6699") # 对应的颜色
)

p <- ComplexHeatmap::Heatmap(
    plot_data,
    top_annotation = top_annotation,
    column_order = rownames(pdata),
    col = color,
    name = "zscore",
    cluster_rows=TRUE,
    cluster_columns=FALSE,
    show_row_names = T,
    show_column_names = T,
    column_title_gp = grid::gpar(fill = c(case_label = hue1[1], control_label = hue[2]))
)
Canton::gs(p,name='热图-532',outdir = outdir,w=16,h=10)
```