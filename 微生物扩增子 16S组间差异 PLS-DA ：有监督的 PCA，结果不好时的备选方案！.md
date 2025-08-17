
做微生物 16S 分析时，你是否遇到过这样的场景：

用 PCA 分析样本分群，结果图上各组样本像 “一锅粥” 混在一起，PC1 和 PC2 的解释率加起来还不到 30%？

这时不妨试试 **PLS-DA**—— 这个被称为 “有监督的 PCA” 的分析方法，或许能帮你拨开数据迷雾！

## 一、PCA 不行？试试 PLS-DA 的 “定向突围”

PCA（主成分分析）是微生物数据分析的 “常客”，但它有个短板：**无监督性**。

它只关注数据中最大的变异信息，不管样本的分组标签（比如 “对照组” 和 “处理组”）。如果组间差异信号被样本内的随机变异掩盖，PCA 就很难呈现清晰的分群。

而 PLS-DA（偏最小二乘判别分析）是一种 **有监督的降维方法**：

它会主动利用样本的分组信息（如 Group 标签），优先提取能区分不同组别的变异成分，让组间差异更明显。

简单说：PCA 是 “让数据自己说话”，PLS-DA 是 “引导数据说出我们关心的话”。

## 二、实战代码：从数据处理到 PLS-DA 可视化

下面用microeco包演示 16S 数据的 PLS-DA 分析全过程，附详细注释：

```
# 加载必要的R包

library(microeco)  # 微生物组分析工具包
library(Canton)    # 绘图辅助包

# 加载示例数据（可替换为你的OTU表）
data(dataset)
mt <- dataset  # mt为核心数据对象

# 1. 数据预处理：规范分类信息
mt %<>% tidy_taxonomy()  # 统一分类级别格式（如k__Kingdom）
mt$tidy_dataset()        # 确保OTU表、样本信息、分类注释一致

# 2. 过滤非目标物种和污染序列
# 只保留细菌和古菌（排除真核生物等）
mt$tax_table %<>% subset(Kingdom == "k__Archaea" | Kingdom == "k__Bacteria")

# 去除线粒体和叶绿体序列（16S分析中常见污染）
mt$filter_pollution(taxa = c("mitochondria", "chloroplast"))

# 再次整理数据，确保过滤后信息一致
mt$tidy_dataset()
mt %<>% tidy_taxonomy()
  
# 3. 标准化处理：稀释曲线至相同测序深度
mt$cal_abund(rel = TRUE)  # 计算相对丰度（可选）
mt_rarefied <- clone(mt)  # 复制对象用于稀释
# 稀释至每个样本1000条序列（消除测序深度差异影响）
mt_rarefied$rarefy_samples(sample.size = 1000)

# 4. 计算beta多样性（ Bray-Curtis距离，用于后续分析）
mt_rarefied$cal_betadiv(method = "bray")  

# 5. 构建PLS-DA模型
# 基于Bray距离创建beta多样性转换对象，指定分组列"Group"
t1 <- trans_beta$new(dataset = mt_rarefied, group = "Group", measure = "bray")
# 计算PLS-DA：提取2个预测成分（predI=2），无正交成分（orthoI=0）
t1$cal_ordination(method = "PLS-DA", predI = 2, orthoI = 0)

# 6. 可视化PLS-DA结果
# 绘制散点图+组内椭圆，按Group着色
p1 <- t1$plot_ordination(plot_color = "Group",
 plot_type = c("point", "ellipse"),
color_values = hue)  # hue为自定义颜色向量

# 美化坐标轴标签（显示成分解释率）
x <- p1$labels$x %>%
  str_replace('p1', 'Component 1: ') %>%  # 将p1替换为"主成分1"
  str_remove_all('(\\[)|(\\])')           # 去除括号

y <- p1$labels$y %>%
  str_replace('p2', 'Component 2: ') %>%  # 同理处理p2
  str_remove_all('(\\[)|(\\])')

# 最终绘图：添加标题、调整主题
p2 <- p1 +
  labs(title = 'PLS-DA on OTU level', x = x, y = y) +
  theme_bw(base_size = 18) +  # 简洁白背景主题
  theme(plot.title = element_text(hjust = 0.5))  # 标题居中

# 保存图片
Canton::gs(p2, outdir = outdir, name = "PLS-DA", w = 8, h = 7)
```

![image.png](https://s2.loli.net/2025/08/17/bzUFwpl9D7Lf8Au.png)
- 这里我用自带的数据集演示 效果并没有太好 此外还可以尝试'PCoA', 'NMDS', 'PCA', 'DCA',  'OPLS-DA'方法但是注意'OPLS-DA'只用于两组数据分析
- `orthoI = 0`是`PLS-DA`模型下的默认参数，`predI = 2` ，ropls包的这个参数默认值是NA，理解起来也比较复杂。如果不指定的话再自带数据上没报错，但是在我自己的数据上报错

## 三、结果解读：3 个关键点

1. **坐标轴含义**：
横轴（Component 1）和纵轴（Component 2）分别代表能区分组别的第 1、第 2 主成分，括号内的数值是该成分解释的组间变异比例（类似 PCA 的解释率）。

2. **样本点与椭圆**：
同组样本的点会尽量聚集，椭圆表示组内样本的 95% 置信区间。如果椭圆不重叠，说明组间差异显著。

3. **与 PCA 的对比**：

若 PCA 中组间混杂，但 PLS-DA 中分离明显，说明分组信息确实能帮助提取关键变异（这正是 PLS-DA 的价值！）。

### 四、什么时候用 PLS-DA？

1. PCA 结果不理想（组间分离差），但你有明确的分组信息（如处理组 / 对照组、疾病 / 健康）；
2. 样本量较少（PLS-DA 对小样本的耐受性优于某些统计方法）；
3. 想聚焦于 “分组相关的差异” 而非 “所有变异”（比如筛选与疾病相关的菌群特征）。

## 总结

PLS-DA 不是 PCA 的 “替代品”，而是 “互补方案”—— 当你需要利用分组信息强化组间差异时，它能成为破解数据僵局的利器。

下次分析 16S 数据时，若 PCA 结果让你失望，不妨试试这份代码，或许会有新发现！

（代码可直接复制使用，只需替换你的 OTU 表和样本信息哦～）

## 参考
```R
function (method = "PCoA", ncomp = 2, taxa_level = NULL, NMDS_matrix = TRUE,
    trans = FALSE, scale_species = FALSE, scale_species_ratio = 0.8,
    orthoI = NA, ordination = deprecated(), ...)
{
    if (lifecycle::is_present(ordination)) {
        lifecycle::deprecate_warn("1.8.0", "cal_ordination(ordination)",
            "cal_ordination(method)")
        method <- ordination
    }
    if (is.null(method)) {
        stop("Input method should not be NULL !")
    }
    if (!method %in% c("PCA", "PCoA", "NMDS", "DCA", "PLS-DA",
        "OPLS-DA")) {
        stop("Input method should be one of 'PCoA', 'NMDS', 'PCA', 'DCA', 'PLS-DA' and 'OPLS-DA' !")

    }
    use_data <- self$dataset
    if (!is.null(taxa_level)) {
        check_tax_level(taxa_level, use_data)
        use_data <- use_data$merge_taxa(taxa_level)
    }

    if (method %in% c("PCA", "DCA", "PLS-DA", "OPLS-DA")) {
        plot.x <- switch(method, PCA = "PC1", DCA = "DCA1", `PLS-DA` = "p1",
            `OPLS-DA` = "p1")
        plot.y <- switch(method, PCA = "PC2", DCA = "DCA2", `PLS-DA` = "p2",
            `OPLS-DA` = "o1")
        if (trans == T) {
            abund <- sqrt(use_data$otu_table)
        }
        else {
            abund <- use_data$otu_table
        }
        abund %<>% t
        if (method == "PCA") {
            model <- rda(abund, ...)
            expla <- round(model$CA$eig/model$CA$tot.chi * 100, 1)
        }
        if (method == "DCA") {
            model <- decorana(abund, ...)
            expla <- round(eigenvals(model)/model$totchi * 100, 1)
        }
        if (method %in% c("PLS-DA", "OPLS-DA")) {
            if (is.null(self$group)) {
                stop("For the current method, group is necessary! Please recreate the trans_beta object and provide the group parameter!")
            }
            use_group <- self$sample_table[, self$group]
            if (method == "PLS-DA") {
                model <- ropls::opls(abund, use_group, orthoI = 0,
                  ...)
            }
            else {
                model <- ropls::opls(abund, use_group, orthoI = orthoI,
                  ...)
            }
            expla <- model@modelDF[, "R2X"] * 100
            names(expla) <- rownames(model@modelDF)
        }
        if (method %in% c("PCA", "DCA")) {
            scores_sites <- scores(model, choices = 1:ncomp,
               display = "sites")
        }
        else {
            if (method == "PLS-DA") {
                model_score <- model@scoreMN
            }
            else {
                model_score <- cbind.data.frame(model@scoreMN,
                  model@orthoScoreMN)
            }
            scores_sites <- model_score[, 1:ncomp, drop = FALSE]
        }
        combined <- cbind.data.frame(scores_sites, use_data$sample_table[rownames(scores_sites), , drop = FALSE])
        if (method %in% c("PCA", "DCA")) {
            loading <- scores(model, choices = 1:ncomp, display = "species")
        }
        else {
            if (method == "PLS-DA") {
                model_loading <- model@loadingMN
            }
            else {
                model_loading <- cbind.data.frame(model@loadingMN,
                  model@orthoLoadingMN)
            }
            loading <- model_loading[, 1:ncomp, drop = FALSE]
        }
        loading %<>% as.data.frame
        if (scale_species) {
            for (i in 1:ncomp) {
               maxratio <- max(abs(scores_sites[, i]))/max(abs(loading[,
                  i]))
                loading[, i] %<>% {
                  . * maxratio * scale_species_ratio
                }
           }
        }
        loading[, "dist"] <- apply(loading, 1, function(x) {
            sum(x^2)
        })
        loading <- loading[order(loading[, "dist"], decreasing = TRUE), ]
        outlist <- list(model = model, scores = combined, loading = loading, eig = expla)
    }
    if (method == "PCoA") {
        if (is.null(self$use_matrix)) {
            stop("Please recreate the object and set the parameter measure !")
        }
        model <- ape::pcoa(as.dist(self$use_matrix), ...)
        combined <- cbind.data.frame(model$vectors[, 1:ncomp],
            use_data$sample_table)
        colnames(combined)[1:ncomp] <- paste0("PCo", 1:ncomp)
        expla <- round(model$values[, 1]/sum(model$values[, 1]) * 100, 1)
        names(expla) <- paste0("PCo", 1:length(expla))
        outlist <- list(model = model, scores = combined, eig = expla)
    }
    if (method == "NMDS") {
        if (NMDS_matrix) {
            if (is.null(self$use_matrix)) {
                stop("Please recreate the object and set the parameter measure !")
            }
            model <- vegan::metaMDS(as.dist(self$use_matrix), k = ncomp, ...)
        }
        else {
            abund <- use_data$otu_table %>% t
            model <- vegan::metaMDS(abund, k = ncomp, ...)
        }
        combined <- cbind.data.frame(model$points, use_data$sample_table)
        outlist <- list(model = model, scores = combined)
        if (!NMDS_matrix) {
            loading <- scores(model, choices = 1:ncomp, display = "species") %>%          as.data.frame

            if (scale_species) {

                for (i in 1:ncomp) {

                  maxratio <- max(abs(combined[, i]))/max(abs(loading[,

                    i]))

                  loading[, i] %<>% {

                    . * maxratio * scale_species_ratio

                  }

                }

            }

            loading[, "dist"] <- apply(loading, 1, function(x) {

                sum(x^2)

            })

            loading <- loading[order(loading[, "dist"], decreasing = TRUE),

                ]

            outlist$loading <- loading

        }

    }

    outlist$ncomp <- ncomp

    self$res_ordination <- outlist

    if (!is.null(taxa_level)) {

        self$res_ordination$taxa_level <- taxa_level

    }

    message("The result is stored in object$res_ordination ...")

    self$ordination_method <- method

    invisible(self)

}
```