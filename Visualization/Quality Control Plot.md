![](https://cdn.nlark.com/yuque/0/2024/png/975582/1725008550054-1dd0544c-f537-4ac4-be54-362fa19a3080.png)

```
#' @TODO 表达矩阵质量控制图
#' @title
#' @description PCA、聚类图、热图
#' @param data 矩阵，行基因，列样本
#' @param group vector 用于PCA分组
#' @param orders heatmap 样本顺序
#' @param od 向量，结果输出路径
#' @param prefix 字符串，用于结果文件的命名
#' @param savePlot
#' @return 拼图
#' @examples v_QCPlot(data, group, orders, prefix = cellinle, od = od)
#' @author *CY*
#'

v_QCPlot <- function(data, group, orders, prefix, od = getwd(), savePlot = TRUE) {
    # 筛选高变SD基因
    v_sd_filter <- function(data, top = 20) {
        tops <- apply(data, 1, sd) %>%
            sort(decreasing = T) %>%
            .[1:top] %>%
            names()
        return(data[tops, ])
    }

    # PCA plot
    library(FactoMineR)
    library(factoextra)
    pca <- PCA(t(data), scale.unit = TRUE, ncp = 2, graph = F)

    p1 <- fviz_pca_ind(
        X = pca, ## pca对象
        axes = 1:2, ## 展示的两个主成分
        geom = "point", ## 展示individual的形式
        habillage = group, ## individual用来分组的变量
        legend.title = "Groups", ## 分组变量的title
        palette = "lancet", ## 颜色面板
        addEllipses = T, ## 是否绘制椭圆
        ellipse.level = 0.8, ## 椭圆的大小
        title = "PCA Plot", ## 标题
        mean.point = T ## 不删除每个组的重心
    ) +
        theme(plot.title = element_text(hjust = 0.5, size = 20))

    # 第二张图：聚类树图
    library(ggdendro)

    p2 <- v_sd_filter(data, 5000) %>%
        t() %>%
        dist() %>%
        hclust() %>%
        ggdendrogram()

    # 第三张图：聚类热图

    library(ComplexHeatmap)

    p3 <- Heatmap(v_sd_filter(data, 20) %>% scale(),
        cluster_rows = TRUE,
        cluster_columns = FALSE,
        column_order = orders # 样品顺序
        , row_order = NULL,
        column_title = "",
        show_heatmap_legend = TRUE,
        heatmap_legend_param = list(title = prefix)
    )
    # 拼图
    library(patchwork)

    p123 <- (p1 + p2) / wrap_elements(plot = grid.grabExpr(draw(p3)), clip = T) +
        plot_layout(heights = c(1, 1)) + plot_annotation(tag_levels = "A")

    if (savePlot) ggsave(file = str_glue("{od}/{prefix}_QCplot.pdf"), plot = p123)
    return(p123)
}
```

# References