CibersortX网站是常用的工具，但是是网页上传数据，现在网页503打不开，而BayesPrism在PMID: [37717006](https://pubmed.ncbi.nlm.nih.gov/37717006) 文章benchmark 9种方法中发现BayesPrism的假阳性与假阴性数量上最低，并且在分解精细的免疫谱系时展现出最佳性能；因此可以作为替代工具，并且BayesPrism也提供了网页工具，不过我们还是习惯本地跑代码；新版的BayesPrism支持系数矩阵作为输入。

BayesPrism是一个综合工具，旨在利用贝叶斯统计方法从bulk RNA测序数据中精确解析肿瘤微环境的细胞组成，并同时考虑细胞特异性的基因表达模式，通过先进的算法模块实现对复杂细胞混合物的深入分析和理解。

BayesPrism包含细胞去卷积模块和嵌入学习模块。细胞去卷积模块依据来自单细胞RNA测序（scRNA-seq）的细胞类型特异性表达轮廓建立先验，联合估计肿瘤（或非肿瘤）样本的bulk RNA-seq表达数据中细胞类型组成及其特异性基因表达的后验分布。嵌入学习模块采用期望最大化算法，基于去卷积模块推测出的非恶性细胞表达量和比例条件，通过线性组合恶性基因程序来近似肿瘤表达模式。

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1721531020278-8edeace1-03d2-4f1c-bf08-ae993d9e4001.png "BayesPrism示意图，每个步骤在做什么")

## 安装

```
library(remotes)
remotes::install_github("Danko-Lab/BayesPrism")
```

注：using是我写的函数，有需要可以后台联系，加入交流群；using作用是一次性加载多个R包，不用写双引号，并且不在屏幕上打印包的加载信息

加载示例数据，可以后台联系获得数据代码和结果文件

```
using(remotes, data.table, BayesPrism)
load('tutorial.gbm.rdata')
```

## 输入文件

`bk.dat`: bulk转录组表达谱（矩阵、行样本、列基因）

`sc.dat`: 单细胞转录组表达谱（矩阵、行样本、列基因）

`cell.type.labels`: 细胞标签（向量）

`cell.state.labels`: 细胞状态标签（向量，每个标签对应的细胞数目要大于20）

注意：

1. bk.dat等只是变量名换成其他也行；
2. bk.dat和sc.dat为同样的标准化方式，支持raw count、TPM, RPM, RPKM, FPKM，不支持log转化后的数据

## 质控图

相关性图查看样本间相关性

```
BayesPrism::plot.cor.phi(
    input = sc.dat,
    input.labels = cell.state.labels,
    title = "cell state correlation",
    pdf.prefix="gbm.cor.cs",
    cexRow = 0.2,
    cexCol = 0.2,
    margins = c(2, 2)
)
```

相关性图查看细胞类型间相关性

```
BayesPrism::plot.cor.phi(
    input = sc.dat,
    input.labels = cell.type.labels,
    title = "cell type correlation",
    pdf.prefix="gbm.cor.ct",
    cexRow = 0.5,
    cexCol = 0.5
)
```

## 过滤基因

目的：去除线粒体、核糖体基因、性染色体基因、低表达基因，只选择编码蛋白的基因

绘制单细胞和bulk的离群基因

图中显示每个基因的归一化平均表达(x 轴)和最大特异性(y 轴)的对数，以及每个基因是否属于一个潜在的“异常”，糖体蛋白基因通常表现出高平均表达水平和低细胞类型特异性得分。

```
sc.stat <- BayesPrism::plot.scRNA.outlier(
    input = sc.dat, # make sure the colnames are gene symbol or ENSMEBL ID
    cell.type.labels = cell.type.labels,
    species = "hs", # currently only human(hs) and mouse(mm) annotations are supported
    return.raw = TRUE # return the data used for plotting.
    pdf.prefix="gbm.sc.stat"
)
```

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1721533486813-357194a4-4214-4c0d-abd2-8e98220dcfe7.png)

核

```
bk.stat <- BayesPrism::plot.bulk.outlier(
    bulk.input = bk.dat, # make sure the colnames are gene symbol or ENSMEBL ID
    sc.input = sc.dat, # make sure the colnames are gene symbol or ENSMEBL ID
    cell.type.labels = cell.type.labels,
    species = "hs", # currently only human(hs) and mouse(mm) annotations are supported
    return.raw = TRUE
    pdf.prefix="gbm.bk.stat"
)
```

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1721533501125-7735ce2f-58c8-420f-a1f7-18bce233c032.png)

去除线粒体、核糖体基因、性染色体基因、低表达基因

```
sc.dat.filtered <- BayesPrism::cleanup.genes(
    input = sc.dat,
    input.type = "count.matrix",
    species = "hs",
    gene.group = c("Rb", "Mrp", "other_Rb", "chrM", "MALAT1", "chrX", "chrY"),
    exp.cells = 5
)
```

绘制bk.dat与sc.dat间的相关性（只支持人的基因数据）

```
BayesPrism::plot.bulk.vs.sc(
    sc.input = sc.dat.filtered,
    bulk.input = bk.dat
    pdf.prefix="gbm.bk.vs.sc"
)
```

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1721533563535-fab6365f-cf51-41ce-b71a-65946da8677d.png)

只选择编码蛋白的基因

```
sc.dat.filtered.pc <- BayesPrism::select.gene.type(sc.dat.filtered, gene.type = "protein_coding")
```

## 选择signature基因

用差异分析方法给每个细胞类型选择相应的marker基因（>50），如果基因少，可以调整阈值

```
diff.exp.stat <- BayesPrism::get.exp.stat(
    sc.dat = sc.dat[, colSums(sc.dat > 0) > 3], # filter genes to reduce memory use
    cell.type.labels = cell.type.labels,
    cell.state.labels = cell.state.labels,
    pseudo.count = 0.1, # a numeric value used for log2 transformation. =0.1 for 10x data, =10 for smart-seq. Default=0.1.
    cell.count.cutoff = 50, # a numeric value to exclude cell state with number of cells fewer than this value for t test. Default=50.
    n.cores = 1 # number of threads
)
```

```
sc.dat.filtered.pc.sig <- BayesPrism::select.marker(
    sc.dat = sc.dat.filtered.pc,
    stat = diff.exp.stat,
    pval.max = 0.01,
    lfc.min = 0.1
)
```

## 构造Prism对象

```
myPrism <- BayesPrism::new.prism(
    reference = sc.dat.filtered.pc,
    mixture = bk.dat,
    input.type = "count.matrix",
    cell.type.labels = cell.type.labels,
    cell.state.labels = cell.state.labels,
    key = "tumor",
    outlier.cut = 0.01,
    outlier.fraction = 0.1,
)
```

## 运行BayesPrism

分布运行，

```
bp.res.initial <- BayesPrism::run.prism(prism = myPrism, n.cores = 50, update.gibbs = FALSE)
base::saveRDS(bp.res.initial, file = file.path(outdir, "bp.res.initial.rds"))
bp.res.update <- BayesPrism::update.theta(bp = bp.res.initial)
base::saveRDS(bp.res.update, file = file.path(outdir, "bp.res.update.rds"))
```

## 提取细胞比例

BayesPrism在输出中同时保留了细胞类型组成θ0的初始估计值和经过更新的细胞类型组成估计值θf。大多数情况下，用户应使用更新后的θ值，因为它往往能改进初始估计。然而，在某些特殊情况下，可能需要使用初始估计值θ0。例如，当混合物中肿瘤成分含量较少（<50%）时，或者参考样本和混合样本之间不存在批次效应，比如参考是从同一bulk RNA-seq平台上通过流式细胞分选获得的情况下。

```
theta <- BayesPrism::get.fraction(
    bp = bp.res.update,
    which.theta = "final",
    state.or.type = "type"
)
data.table::as.data.table(theta, keep.rownames = "Sample") %>% data.table::fwrite(file.path(outdir, "theta.csv"))
```

## Reference

```
https://www.bayesprism.org/
https://github.com/Danko-Lab/BayesPrism
```