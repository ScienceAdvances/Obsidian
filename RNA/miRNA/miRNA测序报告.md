## 1. 背景介绍

微小 RNA（miRNAs）是一类小型 RNA 分子，其长度约为 22 个核苷酸。它们通过与信使 RNA（mRNA）的 3′ - 非翻译区（3′ - UTR）碱基配对，在 mRNA 的翻译调控和降解过程中发挥重要作用。miRNAs 来源于长度约为 70 - 120 个核苷酸的前体转录本，这些前体转录本能够折叠形成茎环结构，并且在基因组进化过程中被认为是高度保守的。在所有人类基因中约有 1%是 miRNA 基因，它们调控着超过 10%的人类编码基因的蛋白质合成[1] 。

## 2. 项目流程

从 RNA 样品到最终数据获得，样品检测、建库、测序每一个环节都会对数据质量和数量产生影响，而数据质量又会直接影响后续信息分析的结果。为了从源头上保证测序数据的准确性、可靠性，我们对样品检测、建库、测序每一个生产步骤都严格把控，从根本上确保了高质量数据的产出：

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284757557-3c4df93d-455f-42d2-984a-6c1d5495255d.png)

图2.1 miRNA测序流程图

### 2.1 总RNA样本提取与检测

对 RNA 样品的检测主要包括 4 种方法：

(1)琼脂糖凝胶电泳分析 RNA 降解程度以及是否有污染。

(2)Nanodrop 检测 RNA 的纯度（OD260/280 比值）。

(3)Qubit 对 RNA 浓度进行精确定量。

(4)Agilent 2100 精确检测 RNA 的完整。

### 2.2 文库构建

样品检测合格后，使用 Small RNA Sample Pre Kit 构建文库，利用 Small RNA 的 3’及 5’端特殊结构（5’端有完整的磷酸基团，3’端有羟基），以 total RNA 为起始样品，直接将 Small RNA 两端加上接头，然后反转录合成 cDNA。随后经过 PCR 扩增，PAGE胶电泳分离目标 DNA 片段，切胶回收得到的即为 cDNA 文库。 构建原理图如下：

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284757725-78f519a9-a527-4762-ae3c-de1fe3c66c4a.png)

图2.2 miRNA文库构建原理图

### 2.3 文库构建与质检

文库构建完成后，先使用 Qubit2.0 进行初步定量，稀释文库至 1ng/ul，随后使用Agilent 2100 对文库的插入片段大小进行检测，并使用qPCR对文库的有效浓度进行准确定量（文库有效浓度 ＞2nM），以保证文库质量。

### 2.4 上机测序

库检合格后，把不同文库按照有效浓度及目标下机数据量的需求合并后进行illumina SE50单端测序。

## 3 生物信息分析流程

分析流程示意图如下:

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284757469-8c9dae42-75f9-4892-9525-cd02e0526291.png)

图3.1 分析流程图

### 3.1 原始与过滤数据质量控制

本项目测序数据原始数据和质量控制后的情况见结果文件夹结构:

1-QC/

----[样本编号]_fastp.html <每个样本的质控前后结果报告>

----multiqc_report.html <所有样本的质控前后结果报告>

#### 3.1.1 测序数据说明

测序得到的原始数据转化为序列数据，以fastq格式保存，FASTQ文件是用于存储测序的核苷酸序列和其质量信息的测序生成的标准格式文件，它是序列格式中常见的一种。其核苷酸序列及其测序质量信息都是以 ASCII 码编码的，最初是由一代测序 sanger开发的，目的是为了将FASTA序列与其质量信息合并为一个文件方便研究者查阅，现如今已成为高通量测序数据结果的标准。FASTQ 文件是以四行为一个基本单元，并对应待测样本的其中一条测序信息， 如图所示：

![](https://cdn.nlark.com/yuque/0/2025/png/975582/1736237673329-57f28258-0c5e-4cdc-8f70-37ac456ecc9f.png)

图3.1.1 测序fastq文件格式

上述文件中第一行以“@”开头，随后为Illumina测序标识符(Squence Identifiers)和描述文字；第二行是测序片段的碱基序列；第三行以“+”开头，随后为Illumina测序标识符(也可为空)；第四行是测序片段每个碱基相对应的测序质量值，该行每个字符对应的ASCII值减去33，即为该碱基的测序质量值。

#### 3.1.2 测序质量评估

测序过程本身存在机器错误的可能性，测序错误率分布检查可以反映测序数据的质量，序列信息中每个碱基的测序质量值保存在fastq文件中。如果测序错误率用e表示，Illumina的碱基质量值用Qphred表示，则有：Qphred=-10log10(e)。Illumina Casava 1.8版本碱基识别与Phred分值之间的简明对应关系见下表。

表3.1 Illumina Casava 1.8版本碱基识别与Phred分值之间的简明对应关系

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284757801-067f63cd-09b3-43ef-92f4-4b996f9acfa2.png)

样本测序质量碱基得分如下图。其中横轴为碱基位置，纵轴为对应碱基质量值。

![](https://cdn.nlark.com/yuque/0/2025/png/975582/1736237792674-c43a6b4f-d798-4c2c-acf9-026f2f623cea.png)

图3.1.2 测序碱基质量分布图

测序错误率与碱基质量有关，受测序仪本身、测序试剂、样品等多个因素共同影响。对于 RNA-seq 技术，测序错误率分布具有两个特点：

(1) 测序错误率会随着测序序列（Sequenced Reads）长度的增加而升高，这是由于测序过程中化学试剂的消耗而导致的，并且为Illumina高通量测序平台都具有的特征[6]。

(2) 前几个碱基的位置也会发生较高的测序错误率，是因为在RNA-seq建库过程中反转录需要随机引物，推测前几个碱基测序错误率较高的原因是随机引物碱基和RNA模版的不完全结合[7]。

测序错误率分布检查用于检测在测序长度范围内，有无测序错误率异常的碱基位置。一般情况下，每个碱基位置的测序错误率都应该低于0.5%。

![](https://cdn.nlark.com/yuque/0/2025/png/975582/1736238030114-33b671b1-6c6d-41f5-b984-9fb2a719edba.png)

每个样本过滤掉的read（示例图）

### 3.2 长度与序列来源分析

序列长度分布步骤结果文件夹结构:

--[样本编号]Read_Length_distribution.pdf

--[样本编号]_readlength_dis.txt

#### 3.2.1 样本序列长度分布

一般来说，动物sRNA的长度区间为 18~35nt，植物sRNA的长度区间为 18~30nt，长度分布的峰能帮助我们判断sRNA 的种类，如miRNA集中在22nt，siRNA集中在24nt等。长度分布结果如显示多数样本的序列在21~22nt出现了峰值，表明数据较为可靠。对各样品的过滤后序列的长度分布统计图如下。其中横坐标为序列最大碱基位置， 纵坐标为对应序列个数

![](https://cdn.nlark.com/yuque/0/2025/png/975582/1736308330014-23490541-415f-4e3e-ba0b-0cf95f712f9b.png)

图3.2.1 样本过滤后序列长度分布图

### 3.3 已知miRNA分布

已知与新miRNA鉴定,分布,二级结构预测分析步骤结果文件夹结构:

--miRNA_results_pdfs

----[miRNA名称].pdf

--result.html

--expression.html

--miRNAs_expressed_all_samples.txt

本项目已知miRNA鉴定,分布,二级结构预测情况见结果文件[customer_files/miRNApipeline_miRNA_dis_analysis](https://bioincloud.tech/cloudir/reports/an_miRNA/customer_files/miRNApipeline_miRNA_dis_analysis/)

#### 3.3.1 已知miRNA分布

在预测动物 miRNA 时，广泛使用Blake C.Meyers等人提出的一组注释动物miRNA的标准[8，9]。动物中的miRNA具有保守性，有些保守miRNA可能在一个物种中具有多个旁系同源物，利用miRNA的保守性和上述的注释标准，来分析动物中已知的miRNA。

下图为miRNA茎环结构图，其中上侧臂是5’端，下侧臂是3’端。红色字母代表成熟miRNA。上下字母间的短线是碱基的匹配，突起的环则代表碱基不匹配。

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284759151-59cfefd3-2f19-46d9-a2c0-546c5b206502.png)

图3.3.1 miRNA茎环结构图

下图为对应结构的序列丰度分布图，横坐标代表碱基位置。纵坐标代表检测到的序列频率百分比。红色即代表该条 miRNA的起始位置以及所对应的丰度。

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284759175-a97f326d-896c-43a9-8d9d-9cf1e6296ad0.png)

图3.3.2 miRNA丰度分布图

下图为对应结构的序列丰度详情，括号代表3’端和5’端匹配，点号代表3’端和5’端失配。尾部的两个数字分别是序列数以及错配个数

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284759610-a702969a-dd90-4b77-ae40-1a71d9b80228.png)

图3.3.3 miRNA序列分布详情

各样本已知miRNA鉴定与表达情况统计表如下，其中每列分别代表：

(1)tag_id：序列名称。

(2)miRDeep2 score：比对到已知结构上的分数。

(3)estimated probability that the miRNA is a true positive：miRNA 为真阳性的估计概率。

(4)predicted mature seq. in accordance with miRBase mature seq：序列与miRBase注释的成熟序列重叠，用"TRUE"表示。如果预测的STAR比对序列与miRBase注释的成熟序列重叠，则用"STAR"表示。

(5)total read count：预测到的miRNA，loop 以及 star miRNAs 的读数总和。

(6)mature read count：映射到预测成熟miRNA上的序列数目，包括上游2nt和下游5nt。

(7)loop read count：映射到预测的miRNA loop的序列数目。包括上游2nt和下游5nt。

(8)star read count：映射到预测的miRNA star序列数目包括上游2nt和下游5nt.

(9)significant randfold p-value：预测的miRNA准确度的显著性。

(10)mature miRBase miRNA：预测的已知miRNA的名称。

表3.3.1 miRNA比对情况统计表

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284759217-1ddd6d0e-17b6-4252-9df6-bb90a0292efb.png)

### 3.5 新miRNA预测分析

已知与新miRNA鉴定,分布,二级结构预测分析步骤结果文件夹结构:

--miRNA_results_pdfs

----[miRNA名称].pdf

--result.html

--expression.html

--miRNAs_expressed_all_samples_novel_miRNA.txt

本项目新miRNA鉴定,分布,二级结构预测情况见结果文件[customer_files/miRNApipeline_miRNA_dis_analysis](https://bioincloud.tech/cloudir/reports/an_miRNA/customer_files/miRNApipeline_miRNA_dis_analysis/)

#### 3.5.1新miRNA分布

数据去除保守miRNA、ncRNA、重复序列和已知保守前体比对上的小RNA后，利用前体预测得到非保守的 miRNA。

各样本预测的miRNA鉴定与表达情况统计表如下，其中每列分别代表：

(1)provisional id：序列名称。

(2)miRDeep2 score：比对到已知结构上的分数。

(3)estimated probability that the miRNA is a true positive：miRNA 为真阳性的估计概率。

(4)rfam alert：预测到的miRNA是否与rRNA或tRNA具有序列相似性

(5)total read count：预测到的miRNA，loop 以及 star miRNAs 的读数总和。

(6)mature read count：映射到预测成熟miRNA上的序列数目，包括上游2nt和下游5nt。

(7)loop read count：映射到预测的miRNA loop的序列数目。包括上游2nt和下游5nt。

(8)star read count：映射到预测的miRNA star序列数目包括上游2nt和下游5nt.

(9)significant randfold p-value：预测的miRNA准确度的显著性。

(10)mature miRBase miRNA：预测的已知miRNA的名称。

表3.5.1 miRNA比对情况统计表

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284759691-715d0f3a-41e4-4068-bdf4-dd799b2107f6.png)

### 3.6 新miRNA碱基偏好性分析

新miRNA碱基偏好性分析步骤结果文件夹结构：

--novel_1_22_bar

----[样本编号]-miRNA-1-22NT-bias.png

----[样本编号]-miRNA-1-22NT-bias.txt

--novel_fst_bar

----[样本编号]-miRNA-fst-bias.png

----[样本编号]-novel_fstNT.txt

本项目新碱基偏好性情况见结果文件[customer_files/miRNApipeline_novelmiRNA_bias](https://bioincloud.tech/cloudir/reports/an_miRNA/customer_files/miRNApipeline_novelmiRNA_bias/)

#### 3.6.1新miRNA序列首位碱基偏好性与前22位碱基偏好

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284759957-528a8e5a-1f0a-4a1b-ac3d-3ef021f857c4.png)

图3.6.1 预测的miRNA首位碱基偏好

横坐标为miRNA长度，纵坐标为该长度sRNA中首位碱基出现A/U/C/G的百分比。

### 3.7 定量分析

已知与新miRNA鉴定,分布,二级结构预测分析步骤结果文件夹结构:

--All_miRNA_uniq_rawcounts.txt

本项目miRNA定量情况见结果文件[customer_files/miRNApipeline_Quant](https://bioincloud.tech/cloudir/reports/an_miRNA/customer_files/miRNApipeline_Quant/)

#### 3.7.1 miRNA定量分析

对各样本的miRNA数目进行统计，下表为统计结果：

表3.7.1 miRNA表达水平表

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284760921-7d8b19a0-6e71-44ce-a513-e77778f952ff.png)

### 3.8 PCA分析

PCA主成分分析步骤结果文件夹结构:

--groupPCA_3D.pdf

本项目PCA分析结果见结果文件[customer_files/miRNApipeline_PCA](https://bioincloud.tech/cloudir/reports/an_miRNA/customer_files/miRNApipeline_PCA/)

#### 3.8.1 PCA分析

为了分析研究样本已知miRNA在不同分组中的特异性表达情况，我们对样本不同分组中已知miRNA的丰度进行主成分分析，将样本分组特征进行降维后，选取占比最大的前2个主成分来描述miRNA在不同分组中的特异性表达情况。

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284760732-045e062b-0714-40c3-a2d0-609ccc4609c7.png)

图3.8.2 样本间PCA主成分图

横纵轴分别代表解释差异排名前两位的主成分及其能够解释的变异的百分比，图中点代表样本在两个主成分中的位置，颜色为样本所对应的分组。

### 3.9 差异分析

差异分析步骤结果文件夹结构:

--diff_miRNA_count.csv

--diff_miRNA_down.csv

--diff_miRNA_up.csv

--diff_miRNA_name_down.txt

--diff_miRNA_name.txt

--diff_miRNA_name_up.txt

--diff_miRNA_results.csv

--[分组]_signdiff_miRNA.fa

--[分组]_signdiff_novel_miRNA.fa

--miRNA_all_results.csv

--miRNA_MA.pdf/png/svg

--miRNA_Volcano_plot.pdf/png/svg

--Signdiff_miRNA_heatmap_all.pdf/png/svg

本项目差异分析结果见结果文件[customer_files/miRNApipeline_diffanalysis](https://bioincloud.tech/cloudir/reports/an_miRNA/customer_files/miRNApipeline_diffanalysis/)

#### 3.9.1 差异分析结果

有生物学重复的样品，分析我们采用基于负二项分布的DESeq2进行差异分析[10]，对各个样本分组进行两两之间的比较，找到在不同分组中表达差异的miRNAs。过滤保留p-value小于0.05的差异miRNA作为候选进行靶基因预测:

(1)names:miRNA名称。

(2)baseMean：样本标准化序列数的均值。

(3)log2FoldChange: 处理组与对照组基因表达水平的比值，再经过差异分析软件收缩模型处理，最后以 2 为底取对数。

(4)p.value：显著性p值。

(5)padj：p值经过多重校验校正后的值

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284760657-d9c0b0e4-b6e7-43f0-9ff6-dd67fe880b1d.png)

表3.9.1 miRNA表达差异分析结果

我们对差异分析结果绘制火山图，横坐标代表miRNA在不同实验组中/不同样品中表达倍数变化，纵坐标代表miRNA表达量变化的统计学显著程度，图中的散点代表各个miRNA。

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284761216-36f89453-9da6-477e-b012-95b34d3805b3.png)

图3.9.1 miRNA表达差异分析火山图

X轴表达量，Y轴表示差异倍数，理论上绝大部分miRNA是不显著差异，所以FC峰值位置应位于0附近，点呈正态分布并对称分布。

### 3.10 miRNA靶基因预测与鉴定分析

靶基因预测步骤结果文件夹结构:

--[分组]_allmiRNA_target.txt

--[分组]_allmiRNA_target_up.txt

--[分组]_allmiRNA_target_down.txt

本项目靶基因预测结果见结果文件[customer_files/miRNApipeline_target_predict](https://bioincloud.tech/cloudir/reports/an_miRNA/customer_files/miRNApipeline_target_predict/)

#### 3.10.1 差异miRNA靶基因预测结果

动物中大部分miRNA与靶基因间完全或几乎完全互补配对，并在互补位点的第10或11位核苷酸上发生切割作用以调控靶基因的表达。切割后会产生2个片段，5'剪切片段和3'剪切片段。其中，5'剪切片段包含5'帽子结构和3'羟基;3'剪切片段则包含自由的5'单磷酸和3'polyA尾巴。有参动物miRNA靶基因预测为miRanda、RNAhybrid两个软件的交集，得到miRNA和靶基因间的对应关系。

表3.10.1 差异 miRNA 靶基因预测结果

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284761855-30f063fc-f00e-4b1b-8d7a-32e1b5a4a009.png)

### 3.11 miRNA靶基因GO与KEGG富集分析

富集分析步骤结果文件夹结构:

--all/up/down

----go_bp/cc/mf_dot_plot.pdf/svg

----go_bp/cc/mf_bar_plot.pdf/svg

----go_bp/cc/mf_results.csv

----kegg_bar_plot.pdf/svg

----kegg_dot_plot.pdf/svg

----pathway_results.csv

----[kegg通路].png

本项目富集分析结果见结果文件[customer_files/miRNApipeline_enrich_analysis](https://bioincloud.tech/cloudir/reports/an_miRNA/customer_files/miRNApipeline_enrich_analysis/)

#### 3.11.1 miRNA靶基因GO与KEGG富集分析

我们对差异靶基因进行富集分析。通过富集分析，可以找到不同条件下的差异靶基因与哪些生物学功能或通路显著性相关，从而揭示和理解生物学过程的基本分子机制。

我们采用clusterProfiler软件对差异靶基因集进行GO功能富集分析，KEGG通路富集分析等。富集分析基于超几何分布原理，其中差异靶基因集为miRNA差异分析所得对应靶基因并注释到GO或KEGG数据库的基因集，背景基因集为所有参与差异分析并注释到GO或KEGG数据库的基因集。富集分析结果是对每个差异比较组合的所有差异基因集进行富集。

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284761827-aeab7e98-4fe8-435a-a75a-dc1f8cd00c2b.png)

图3.11.1 GO富集分析原理图

GO(Gene Ontology)是描述基因功能的综合性数据库（http://www.geneontology.org/)，可分为生物过程（Biological Process）和细胞组成（Cellular Component）分子功能（Molecular Function）三个部分。GO 功能显著性富集分析给出与基因集背景相比，在差异表达基因中显著富集的 GO 功能条目，从而给出差异表达基因与哪些生物学功能显著相关。该分析首先把所有差异表达基因向 Gene Ontology 数据库的各个 term 映射，计算每个 term 的基因数目，然后找出与整个基因集背景相比，在差异表达基因中显著富集。GO富集分析方法为clusterProfiler的enricher函数，该方法使用超几何分布检验的方法获得显著富集的GO Term。统计被显著富集的各个GOterm中的基因数，以柱状图的形式展示。GO富集以padj小于0.05为显著富集。

根据GO富集分析结果，在GO的三个分类中（cellular_component:细胞组分；biological_process：生物学过程；molecular_function：分子功能）分别统计每个分类上调基因、下调基因及所有基因的数目，以柱状图展示。

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284762307-f4ab9880-84ca-4d68-aeff-b145da2d9be0.png)

图3.11.2 靶基因GO富集柱状图

在生物体内，不同基因相互协调行使其生物学功能，通过Pathway显著性富集能确定差异表达基因参与的最主要生化代谢途径和信号转导途径。

KEGG(Kyoto Encyclopedia of Genes and Genomes)是有关Pathway的主要公共数据库，其中整合了基因组化学和系统功能信息等众多内容。信息分析时，我们将KEGG数据库按照动物，植物，真菌等进行分类，依据研究物种的种属选择相应的类别进行分析。Pathway显著性富集分析以KEGG Pathway为单位，应用超几何检验，找出差异基因相对于所有有注释的基因显著富集的pathway，KEGG通路富集同样以padj小于0.05作为显著性富集的阈值。

差异靶基因KEGG富集气泡图是KEGG富集分析结果的图形化展示方式。在此图中，KEGG富集程度通过GeneRatio、p.adjust和富集到此通路上的基因个数来衡量。其中GeneRatio指差异表达的基因中位于该pathway条目的基因数目与所有有注释基因中位于该pathway条目的基因总数的比值。p.adjust是做过多重假设检验校正之后的Pvalue，p.adjust的取值范围为[0，1]，越接近于零，表示富集越显著。我们挑选了富集最显著的20条pathway条目在该图中进行展示，若富集的pathway条目不足20条，则全部展示。

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284762128-e6248e80-3790-4035-9fa9-9a814ba8fa93.png)

图3.11.3 KEGG通路富集气泡图

## 4 参考文献

[1] Hsu PW, Huang HD, Hsu SD, Lin LZ, Tsou AP, Tseng CP, Stadler PF, Washietl S, Hofacker IL. miRNAMap: genomic maps of microRNA genes and their target genes in mammalian genomes. Nucleic Acids Res. 2006 Jan 1;34(Database issue):D135-9. doi: 10.1093/nar/gkj135. PMID: 16381831; PMCID: PMC1347497.

[1]Nath， J. and A. Nath， A Modified Algorithm for Quantifying of Pre-mature MiRNAs using Some Fractal Parameters[J]. International Journal of Advanced Computer Research 2012. 2:202-207.

[2]. Bartel DP.MicroRNAs: genomics， biogenesis， mechanism， and function[J].Cell.2004，116(2):281-97.

[3] LONG D， LEE R， WILLIAMS P， et al. Potent effect of target structure on microRNA function [J]. Nature Structural & Molecular Biology， 2007， 14(4): 287-94.

[4].郭韬，李广林，魏强，梁永宏.植物MicroRNA功能的研究进展[J].西北植物学报.2011，31(11):2347-54.

[5]Cock， P.J.A.， Fields， C.J.， Goto， N.， Heuer， M.L.， and Rice， P.M. (2010). The Sanger FASTQ file format for sequences with quality scores， and the Solexa/Illumina FASTQ variants. Nucleic acids research 38， 1767-1771.

[6]Erlich， Y.， and Mitra， P.P. (2008). Alta-Cyclic: a self-optimizing base caller for next-generation sequencing. Nature methods 5， 679-682

[7]Jiang， L.， Schlesinger， F.， Davis， C.A.， Zhang， Y.， Li， R.， Salit， M.， Gingeras， T.R.， and Oliver， B.(2011). Synthetic spike-in standards for RNA-seq experiments. Genome research 21， 1543-1551.

[8]Meyers， B.C.， et al.， Criteria for annotation of plant MicroRNAs[J]. Plant Cell， 2008. 20(12): 3186-90.

[9]Axtell， M.J. and B.C. Meyers， Revisiting Criteria for Plant MicroRNA Annotation in the Eraof Big Data[J]. Plant Cell， 2018. 30(2): 272-284.

[10]Michael I Love，Wolfgang Huber，Simon Anders.(2014).Moderated estimation of fold change and dispersion for RNA-seq data with DESeq2.Genome Biology，DOI:10.1186/s13059-014-0550-8IF: 10.1 Q1 .

[11] German， M.A.， et al.， Construction of Parallel Analysis of RNA Ends (PARE) libraries for the study of cleaved miRNA targets and the RNA degradome[J]. Nat Protoc， 2009. 4(3):356-62.

[12]Young， M.D.， Wakefield， M.J.， Smyth， G.K.， and Oshlack， A. goseq: Gene Ontology testing for RNA-seq datasets.

[13]Kanehisa M， Araki M， Goto S， Hattori M， Hirakawa M， et al. (2008). KEGG for linking genomes to life and the environment. Nucleic Acids research36:D480–484.