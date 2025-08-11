## 1. 背景介绍

非编码RNA（ non-coding RNA，ncRNA ）是细胞中不进行蛋白翻译的 RNA ，ncRNA 包括 rRNA 、 tRNA 、 snRNA 、 snoRNA 、 lncRNA等，这些 RNA 可以与蛋白配合发挥功能也可以独立发挥催化功能。从发现 tRNA 、 rRNA 等开始对具有生物学作用的 ncRNA 的相关研究已有多年历史。

这些 ncRNA 包含丰富的调节和功能单元，其中lncRNA占比较大，即大多数转录但不编码蛋白的序列均与lncRNA相关。长链非编码 RNA（ long non-coding RNA，lncRNA ）是指长度超过 200nt 的 RNA ，其本身不编码蛋白，而是以 RNA 的形式形成多层面调控基因的表达。

目前，已从单细胞真核生物以及人类中鉴定出数以万计的 lncRNA 基因座，并依据其相对于蛋白质编码基因的相对位置，将其分为基因间 lncRNA ，内含子 lncRNA ， 正义 lncRNA ，反义 lncRNA ，双向 lncRNA 五类。

lncRNA 在表观遗传、顺式或反式转录及转录后水平上参与多层次基因表达调控，参与 X 染色体沉默、基因组印记以及染色质修饰、转录激活、转录干扰、核内运输等生物学进程，但经实验鉴定的与疾病相关的 lncRNA 数量仅为已鉴定基因座的1％不到，其生物学功能有待进一步挖掘。

## 2. 项目流程

从 RNA 样品到最终数据获得，样品检测、建库、测序每一个环节都会对数据质量和数量产生影响，而数据质量又会直接影响后续信息分析的结果。为了从源头上保证测序数据的准确性、可靠性，我们对样品检测、建库、测序每一个生产步骤都严格把控，从根本上确保了高质量数据的产出：

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284728790-4872fdcb-7bb8-43a9-8aad-0f372f4792ca.png)

图2.1 lncRNA 项目流程图

### 2.1 总 RNA 样本提取与检测

对 RNA 样品的检测主要包括 4 种方法：

(1)琼脂糖凝胶电泳分析 RNA 完整性。

(2)Nanodrop 检测 RNA 的纯度（OD260/280 比值）。

(3)Qubit 对 RNA 浓度进行精确定量。

(4)Agilent 2100 精确检测 RNA 的完整。

### 2.2 文库构建

RNA 样本检测合格后，使用 Ribo-Zero kit 从总 RNA 样品中去除 rRNA（lncRNA 有些具有与 mRNA 相同的 polyA 尾结构，有些不含有，用去除 rRNA 的方法能够最大程度地保留不含有 polyA 尾的 lncRNA ）。向富集得到的 RNA 中加入 fragmentation buffer 将 RNA 打断成小片段。然后以片段化的 RNA 为模板，加入 6bp 随机引物（ random hexamers ）反转录合成 cDNA 第一条链，再加入缓冲液、 dNTPs（ dNTP 中的 dTTP 用 dUTP 取代）、 DNA polymerase I 和 RNase H 合成 cDNA 第二条链。合成的双链 cRNA 经纯化、末端修复、加 A 、连接测序接头处理后，用 USER 酶降解含有 U 的 cDNA 第二链并进行 PCR 富集，最后用 AMPure XP beads 纯化 PCR 产物，得到最终的链特异性文库。文库构建完成后，先使用 Qubit 2.0 进行初步定量，稀释文库，随后用 Agilent 2100 对文库的插入片段大小进行检测，插入片段符合预期后，使用 qPCR 方法对文库的有效浓度进行准确定量，以保证文库质量。文库构建原理图如下：

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284728737-68fd26fa-be6e-4044-83c8-bd41693f01d7.png)

图2.2 lncRNA 文库构建原理图

### 2.3 库检

文库构建完成后，先使用 Qubit2.0 进行初步定量，稀释文库至 1ng/ul ，随后使用 Agilent 2100 对文库的插入片段大小进行检测，并使用 qPCR 对文库的有效浓度进行准确定量（文库有效浓度 > 2nM ），以保证文库质量。

### 2.4 上机测序

库检合格后，按照有效浓度及目标下机数据量的需求将不同文库混合在一起，使用 Illumina 高通量测序平台进行测序。

## 3 生物信息分析流程

分析流程示意图如下:

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284729045-30f60bb9-07db-481f-be28-53c744bcd110.png)

图3.1 生物信息分析流程图

### 3.1 原始与过滤数据质量控制

原始数据质控步骤结果文件夹结构:

--fastqc

----[样本编号]_fastqc.html [各样本原始数据fastqc软件质控报告]

----[样本编号]_fastqc.zip [各样本原始数据fastqc软件质控文件]

--multiqc

----multiqc_data

------multiqc各项文件

----multiqc_report.html [原始数据multiqc软件质控报告]

--multiqc_report.html

本项目测序数据原始数据质量控制情况见结果文件[lncRNApipeline_rawQC](https://bioincloud.tech/cloudir/reports/an_lncRNA/lncRNApipeline_rawQC/)

过滤数据质控步骤结果文件夹结构:

--fastqc

----[样本编号]_filtered_fastqc.html [各样本过滤数据fastqc软件质控报告]

----[样本编号]_fastqc.zip [各样本过滤数据fastqc软件质控文件]

--multiqc

----multiqc_data

----multiqc_report.html [过滤数据multiqc软件质控报告]

--multiqc_report.html

本项目测序数据过滤数据质量控制情况见结果文件[lncRNApipeline_filterQC](https://bioincloud.tech/cloudir/reports/an_lncRNA/lncRNApipeline_filterQC/)

#### 3.1.1 测序数据说明

测序得到的原始数据转化为序列数据，以 fastq 格式保存， FASTQ 文件是用于存储测序的核苷酸序列和其质量信息的测序生成的标准格式文件，它是序列格式中常见的一种。其核苷酸序列及其测序质量信息都是以 ASCII 码编码的，最初是由一代测序 sanger 开发的，目的是为了将 FASTA 序列与其质量信息合并为一个文件方便研究者查阅，现如今已成为高通量测序数据结果的标准。 FASTQ 文件是以四行为一个基本单元，并对应待测样本的其中一条测序信息， 如下所示：

  
  

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284728944-8bb33d40-46b8-4c19-96b5-4efebbe7e887.png)

  
  

图3.1.1 测序 fastq 文件格式

上述文件中第一行以“ @ ”开头，随后为 Illumina 测序标识符( Squence Identifiers )和描述文字；第二行是测序片段的碱基序列；第三行以“ + ”开头，随后为 Illumina 测序标识符(也可为空)；第四行是测序片段每个碱基相对应的测序质量值，该行每个字符对应的 ASCII 值减去 33 ，即为该碱基的测序质量值。

#### 3.1.2 测序质量评估

测序过程本身存在机器错误的可能性，测序错误率分布检查可以反映测序数据的质量，序列信息中每个碱基的测序质量值保存在 fastq 文件中。如果测序错误率用 e 表示， Illumina 的碱基质量值用 Qphred 表示，则有： Qphred = -10log10(e) 。 Illumina Casava 1.8 版本碱基识别与 Phred 分值之间的简明对应关系见下表。

|   |   |   |   |
|---|---|---|---|
|Phred分值|碱基错误识别概率|碱基正确识别概率|Q-Score|
|10|1/10|90%|Q10|
|20|1/100|99%|Q20|
|30|1/1000|99.9%|Q30|
|40|1/10000|99.99%|Q40|

样本测序质量碱基得分如下图。其中横轴为碱基位置，纵轴为对应碱基质量值。

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284730317-8abb8044-7c32-4f2d-b966-b2fc1b039f89.png)

图3.1.2 测序碱基质量分布图

为了方便看出样品质量概况，用 multiqc 进行合并，如下图：同样其中横轴为碱基位置，纵轴为对应碱基质量值。

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284731492-2b1a23b2-c7af-4636-84d9-1e6f3cf9f858.png)

图3.1.3 多样本测序碱基质量分布图

测序错误率与碱基质量有关，受测序仪本身、测序试剂、样品等多个因素共同影响。对于 RNA-seq 技术，测序错误率分布具有两个特点：

(1) 测序错误率会随着测序序列（ Sequenced Reads ）长度的增加而升高，这是由于测序过程中化学试剂的消耗而导致的，并且为 Illumina 高通量测序平台都具有的特征[6]。

(2) 前几个碱基的位置也会发生较高的测序错误率，是因为在 lncRNA-seq 建库过程中反转录需要随机引物，推测前几个碱基测序错误率较高的原因是随机引物碱基和 RNA 模版的不完全结合[7]。

测序错误率分布检查用于检测在测序长度范围内，有无测序错误率异常的碱基位置。一般情况下，每个碱基位置的测序错误率都应该低于 0.5% 。

#### 3.1.3 各样本原始测序数据质量评估

我们对各样本原始数据的序列数目、碱基数目、 Q20 、 Q30 、 GC 占比等指标进行统计，从而直观的对原始数据大小与质量进行描述并汇总到表3.1.2中。

1. _Sample: 样本编号_
2. _File: 数据文件名称_
3. _Reads: 序列数目_
4. _Bases: 碱基总数_
5. _Q20: 符合 Q20 标准碱基数目占总碱基数的比例_
6. _Q30: 符合 Q30 标准碱基数目占总碱基数的比例_
7. _gc%: GC 百分比_

#### 3.1.3 各样本质控后测序数据质量评估

对整条 read 做保留或弃去处理:

1.当一条 read 中 N 碱基的个数达到一定的数量（默认为 5 个）时，弃去整一条 read 。

2.当一条 read 中低质量碱基（默认质量值低于 15 的碱基为低质量碱基）达到一定的比例（默认这个比例值为 40% ）时，弃去整一条 read 。

3.当一条 read 的碱基个数少于一定的数量（默认为 15 个碱基）时，弃去整一条 read 。

4. adapter 序列裁剪:默认情况下，对 pair-end 的 read1 和 read2 做两序列比对，查看比对结果中， read1 的右端是否会超出 read2 的右端， read2 的左端是否会超出 read1 的左端，如果超出的话，表明是测到了 adapter 序列，则会把超出的部分给剪掉。

我们对各样本过滤后数据的序列数目、碱基数目、 Q20 、 Q30 、 GC 占比等指标进行统计，从而直观的对原始数据大小与质量进行描述并汇总在表3.1.3中。

1. _Sample: 样本编号_
2. _File: 数据文件名称_
3. _Reads: 序列数目_
4. _Bases: 碱基总数_
5. _Q20: 符合 Q20 标准碱基数目占总碱基数的比例_
6. _Q30: 符合 Q30 标准碱基数目占总碱基数的比例_
7. _gc%: GC 百分比_

### 3.2 参考序列比对分析

参考序列比对分析结果文件夹结构:

--[样本编号].bam [比对结果bam文件]

--mapping_stat.csv [比对结果统计表]

--mapping_stat.png [比对结果统计图]

--merged_ac.csv [不同RNA类型统计表]

--rna_type_stat.png [不同RNA类型统计图]

本项目参考序列比对结果文件[lncRNApipeline_mapping](https://bioincloud.tech/cloudir/reports/an_lncRNA/lncRNApipeline_mapping/)

#### 3.2.1 序列比对统计

测序片段是由 RNA 随机打断而来，为了确定这些片段对应基因组上的哪些区域，我们使用 HISAT2 软件将质控后的序列比对到参考基因组上生成.sam或.bam文件，结果文件中包含序列比对到参考基因组的比对情况。

各样本比对情况统计在 表3.2.1 各样本比对情况汇总

1. _Sample: 样本编号_
2. _Overall alignment rate: 总比对率_
3. _Aligned 0 times: 未比对到参考基因组序列数与占比_
4. _Aligned exactly 1 time: 比对一次到参考基因组序列数与占比_
5. _Aligned >1 times: 比对大于一次到参考基因组序列数与占比_

各样本序列比对情况统计图如下。横轴为比对率，纵轴为样本名称，不同的颜色代表过滤数据比对到基因组的次数。

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284731915-018ec7e6-c215-45dd-9e38-95b86a612165.png)

图3.2.1 序列比对情况统计图

#### 3.2.2 样本RNA来源

因为测序文库中除去 rRNA 后还存在各种 RNA 来源于不同类型的分子，因此我们对不同类型的RNA分子进行分类并获取其数目，例如 mRNA ， miRNA ， lncRNA ， repeats ， siRNA 和 piRNA 等。在将读数与不同类型的分子比对后，计算读数和序列分布。文库中应为coding RNA 和 lncRNA为主，如果在测序文库中 RNA 分子类型发生改变，可能是由于实验中测序文库的不良品质引起的。下表是测序文库中不同类型 RNA 分子在各样本中的数目，以反应不同类型 RNA 分子的分布。

1. _Type: RNA分子类型_
2. _列2-列n+1 : 各样本RNA分子数目_
3. _sum: 各类型RNA分子数目总和_

各样本测序文库中的 RNA 来源统计累积柱状图，图横轴为不同类型 RNA 占比，纵轴为样本名称，不同的颜色代表不同的RNA类型，sum为所有样本的RNA类型。

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284731519-bc68a20c-2a02-427a-a48f-a73154abb334.png)

图3.2.2 RNA来源统计图

### 3.3 转录本拼接与注释

转录本拼接与注释结果文件夹结构:

--merged.gtf [转录本组装结果]

--merged_stat.csv [转录本数目统计表]

--merged.annotated.gtf [转录本组装注释结果]

--assemble_trans_type.png [转录本数目统计图]

本项目转录本拼接与注释结果文件[lncRNApipeline_assembles](https://bioincloud.tech/cloudir/reports/an_lncRNA/lncRNApipeline_assembles/)

新转录本预测步骤如下：

1.我们以每个样本的 bam 格式比对结果文件作为输入，使用 StringTie 软件对每个样本单独进行转录本组装（默认参数），得到每个样本的组装转录本基因组注释。

2.合并所有转录本注释，得到一个所有样本的转录本注释。

3.用 gffcompare 软件将该所有样本的转录本注释与参考基因组注释文件比较，提取参考基因组未注释过的转录本，并筛除出预测的新转录本。

StringTie 使用网络流算法以及可选的从头组装来拼接转录本。相对于 cufflinks 等软件，其具有以下优势：1.拼接出更完整的转录本；2.拼接出更准确的转录本；3.更好的估计转录本的表达水平；4.更快的拼接速度。

拼接的转录本的注释信息见 表3.3.1 组装转录本基因组注释表

1. _seqname: 染色体编号_
2. _source: 注释的来源，这里的 StringTie 是指该转录本是由 StringTie软件组装所得_
3. _feature: 注释信息的结构类型，如 gene、transcript、exon 等_
4. _start: 注释区域在染色体上的起始坐标_
5. _end: 注释区域在染色体上的终止坐标_
6. _score: 注释信息的得分，是注释信息可能性的说明，点号表示为空_
7. _strand: 注释区域在染色体上的正负链信息_
8. _frame:仅对注释类型为 CDS 的有效(其它为空)，有效值为0，1，2，分别表示第一个密码子应该从start开始的第1，2，3个碱基算起_
9. _attributes:包含众多属性的列表，主要为基因编号、转录本编号等信息。注意：有的新转录本虽然标注了参考基因ID，但其外显子结构注释与参考基因转录本外显子结构注释差异很大，故判定为新转录本。_

拼接转录本与已知基因比较结果，图横轴为比较后转录本类型，纵轴为转录本数目。鉴定出来的转录本类型如图3.3.2所示，其中参考基因组未注释过的转录本为 i，j，o，u，x 五类转录本。

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284731778-6f60d6dd-2102-4364-9b57-bba8c3f67dd9.png)

图3.3.1 拼接转录本与已知注释基因比较结果柱形图

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284732058-6bc4681f-3577-4f73-aef0-0bb5b7068515.png)

图3.3.2 class code示意图

### 3.4 候选 lncRNA 筛选

候选 lncRNA 筛选结果文件夹结构:

--merged_nc.csv [筛选出的lncRNA信息]

--merged.nc.gtf [筛选出lncRNA注释文件]

--merged.nc.fa [筛选出lncRNA序列文件]

--Merged_PLEK_CPC2_CPAT_results.txt [PLEK，CPC2，CPAT共有预测结果]

--PLEK_results.txt [PLEK共有预测结果]

--CPC2_results.txt [CPC2共有预测结果]

--CPAT_results.txt [CPAT共有预测结果]

--lncrna_pre_venn.png [PLEK，CPC2，CPAT共有预测结果韦恩图]

--lncrna_pre_len.png [筛选出的lncRNA长度分布图]

--lncrna_pre_exon.png [筛选出的lncRNA外显子数目分布图]

本项目候选 lncRNA 筛选结果文件[lncRNApipeline_lncRNAannos](https://bioincloud.tech/cloudir/reports/an_lncRNA/lncRNApipeline_lncRNAannos/)

#### 3.4.1 lncRNA筛选步骤

lncRNA 是一类转录本长度超过 200nt ，不编码蛋白的RNA 分子。我们基于转录组拼接结果，根据 lncRNA 的结构特点以及不编码 lncRNA 的功能特点，设置一系列严格的筛选条件，通过以下 5 步的筛选，将筛选所得 lncRNA 作为最终的候选 lncRNA 集进行后续分析。

1.转录本 exon 个数筛选:过滤转录本拼接结果中大量低表达量，低可信度的单外显子转录本，选择 exon 个数 >=2 的转录本。

2.转录本长度筛选:选择转录本长度 > 200bp 的转录本。

3.转录本已知注释筛选:通过 Cuffcompare 软件，筛除与数据库注释 exon 区域有重叠的转录本，并将数据库中，与本次拼接转录本 exon 区域有重叠的 lncRNA 作为数据库注释 lncRNA 纳入到后续分析。

4.利用 cuffcompare 的结果中 class_code 信息筛选候选 i ， j ， o ， u ， x 这 5 类转录本。

5.编码潜能筛选:是否具有编码潜能是判断转录本是否为 lncRNA 的关键条件。针对拼接得到的转录本，我们综合了目前主流的编码潜能分析方法进行该项筛选，取这些软件分析结果中没有编码潜能的转录本的交集作为本次分析预测得到的 lncRNA 数据集。

#### 3.4.2 编码潜力筛选

具有编码潜能与否是判断转录本是否为 lncRNA 的关键条件，我们综合了目前主流的编码潜能分析方法进行该项筛选，主要包括: CPC 分析、CNCI 分析、CPAT 三种方法，最终的 lncRNA 数据集来自这几种分析方法筛选的交集。

CPC ( Coding Potential Calculator )是一种 lncRNA 编码潜能计算工具，将转录本与已知 lncRNA 数据库做 blastx 比对，依据转录本各个编码框的生物学序列特征，通过支持向量机的分类器来评估转录本的编码潜能。这里的数据库使用的是 NCBI 真核 lncRNA 数据库( NRDB )。前 10 个转录本CPC2编码潜能如下。

1. _ID: 转录本编号_
2. _transcript_length: 转录本长度_
3. _peptide_length: 潜在翻译蛋白序列长度_
4. _Fickett_score: Fickett得分是一个用于评价核酸序列编码潜力的统计分数。_
5. _pl: pl值_
6. _ORF_integrity: ORF完整度_
7. _coding_probability: 编码潜能得分_
8. _label: 转录本编码标签_

CPAT (Coding Potential Accessment Tool) 作为一种新的无比对预测方法， CPAT 可以从大量候选转录本中快速识别编码和非编码转录本，基于以下 4 种序列特征:开放阅读框大小，开放阅读框覆盖， Fickett TESTCODE 统计和六联体使用偏倚，并基于 SVM 构建的逻辑回归模型。前 10 个转录本CPAT编码潜能如下。

1. _seq_ID: 转录本编号_
2. _ID: CPAT软件转录本编号_
3. _mRNA: 转录本长度_
4. _ORF_strand: ORF正负链_
5. _ORF_frame: ORF类型_
6. _ORF_start: ORF的起始位置。_
7. _ORF_end: ORF的终止位置。_
8. _ORF: ORF的长度。_
9. _Fickett: Fickett得分是一个用于评价核酸序列编码潜力的统计分数。_
10. _Hexamer: 这可能表示用于评价ORF中六核苷酸序列使用偏好的分数，与编码潜力有关。_
11. _Coding_prob: lncRNA编码蛋白质潜力。_

PLEK 是一款基于序列的kmer构成来区分编码和非编码转录本，不需要通过比对来完成，所以运行速度较快，同时其性能受到测序错误的影响的概率较低，比较稳定。前 10 个转录本 PLEK 编码潜能如下。

1. _第1列: lncRNA编码标签_
2. _第2列: lncRNA编码蛋白质潜力_
3. _第3列: 转录本编号_

#### 3.4.3 候选 lncRNA 描述性统计

我们选择三款软件PLEK，CPC2，CPAT获得的 lncRNA 作为候选 lncRNA ，并在选择这些 lncRNA 进行后续分析，下图是PLEK，CPC2，CPAT软件预测的候选 lncRNA 韦恩图。每个圈为各个软件获得的潜在 lncRNA 数目，它们的交集为候选 lncRNA 。

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284732213-490b9572-7662-4393-9608-03571de03fca.png)

图3.4.1 软件预测 lncRNA 韦恩图

我们对候选 lncRNA 长度进行统计，得到候选 lncRNA 长度概率密度统计图，横轴为 lncRNA 长度，纵轴为概率密度。

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284732418-8c7a19a2-bd1d-4960-9d22-264dfa4d0a7c.png)

图3.4.2 候选 lncRNA 长度分布统计图

我们对候选 lncRNA 外显子数目进行统计，得到候选 lncRNA 外显子概率密度统计图，横轴为 lncRNA 外显子数目，纵轴为概率密度。

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284732375-e6dea3ec-9d8c-4caa-9bbd-a59c31f1786d.png)

图3.4.3 候选 lncRNA 外显子数目统计图

### 3.5 lncRNA 定量分析

lncRNA定量步骤结果文件夹结构:

--lncRNA_gene_expr.csv [筛选出的lncRNA所在基因的原始序列数目]

--lncRNA_trans_expr.csv [筛选出的lncRNA转录本的原始序列数目]

--Sample_boxplot.png [lncRNA基因表达箱线图]

--Sample_cluster_heatmap.png [样本相关性聚类图]

--PCA.png [样本PCA图]

--PCA_vp.png [主成分个维度占比图]

--Gene_expr_heatmap.png [基因表达热图]

本项目lncRNA定量分析结果见结果文件[lncRNApipeline_lncRNAexp](https://bioincloud.tech/cloudir/reports/an_lncRNA/lncRNApipeline_lncRNAexp/)

#### 3.5.1 定量结果展示

筛选得到 lncRNA 转录本之后，我们对已知基因，转录本以及候选lncRNA 转录本进行定量分析，得到各样本的基因与转录本的定量信息表:

1. _gene_id: 基因名称_
2. _列2-列n+1 : 各样本基因定量结果_
3. _transcript_id: 转录本名称_
4. _列2-列n+1 : 各样本转录本定量结果_

#### 3.5.2 lncRNA表达水平分析

通过所有 lncRNA 的 FPKM 的箱型图对不同实验条件下的表达水平进行比较。横轴为样本，纵轴为表达量，图中的点为表达量分布在1.5IQR之外的基因。

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284732814-59409004-d497-4d22-9da4-8345d724b9d6.png)

图3.5.1 lncRNA 表达箱线图

#### 3.5.2 样本相关性分析

为了反映样本间 lncRNA 表达的相关性，本研究计算了每两个样品之间所有定量 lncRNA 的 Pearson 相关系数( 皮尔逊相关系数 )，并将这些系数以热图的形式反映出来，颜色越深代表相关性越高。

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284733008-a8594a7d-ea59-4240-a691-3b4ef6f04009.png)

图3.5.2 样本Pearson相关性图

#### 3.5.3 PCA 分析

主成分分析（ PCA ）是将多个变量通过降维为少数几个相互独立的变量（即主成分），同时尽可能多地保留原始数据信息的一种多元统计分析方法。

在 lncRNA 组分析中，PCA 将样本所包含的大量 lncRNA 表达量信息降维为少数几个互相无关的主成分，以进行样本间的比较，方便找出离群样品、判别相似性高的样品簇等。较好的分析结果为组内样本聚集在一起，组间样本分离。

我们对样本不同分组中定量 lncRNA 的 FPKM 进行主成分分析，将样本分组特征进行降维后，选取前两个主成分来描述 lncRNA 在不同分组中的特异性表达情况。

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284733289-0e98d1f3-dcae-43e7-8470-8de9e73c9c55.png)

图3.5.3 PCA 主成分各维度占比图

选用前两个主成分进行画PCA图。不同的颜色代表不同的分组，相同颜色点之间的距离代表组内重复度，越集中代表重复性越好，不同颜色点之间的距离代表比较分组之间的差异，距离越大代表分组之间差异越大。

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284733534-8ed9e818-5469-4a9e-af83-3e6e791139a4.png)

图3.5.4 样品分组 PCA 图

#### 3.5.4 层次聚类

表达模式相似的 lncRNA 通常具有功能相关性，我们利用热图，以欧式距离为矩阵计算公式，对样品表达 lncRNA 和样品同时进行层次聚类。层次聚类热图如下所示， x 轴的每个坐标代表每个样本，红色颜色越深代表表达水平越高。

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284733635-d9d04542-e2a5-4c1c-85a4-969b9ac0fb02.png)

图3.5.5 lncRNA 表达量层次聚类图

### 3.6 差异表达分析

差异分析步骤结果文件夹结构:

--lncRNA_allgene_diff_results.csv [差异分析结果(基因)]

--lncRNA_alltrans_diff_results.csv [差异分析结果(转录本)]

--diff_lncRNA_up.csv [显著上调lncRNA所在基因]

--diff_lncRNA_down.csv [显著下调lncRNA所在基因]

--lncRNA_gene_MA.png [差异基因MA图]

--lncRNA_gene_voconol.png [差异基因火山图]

--diff_lncRNA_trans_up.csv [显著上调lncRNA]

--diff_lncRNA_trans_down.csv [显著下调lncRNA]

--lncRNA_trans_MA.png [差异转录本MA图]

--lncRNA_trans_voconol.png [差异转录本火山图]

本项目差异分析结果见结果文件[lncRNApipeline_diffanalysis](https://bioincloud.tech/cloudir/reports/an_lncRNA/lncRNApipeline_diffanalysis/)

#### 3.6.1 差异分析结果

lncRNA 差异性分析能够极大地推动发现新的生物标志物，提升生物标志物鉴定的精准度，对分子作用机理、生物标志物、疾病早诊、分子分型、预后以及临床诊断等均具有重要价值。

lncRNA 差异表达的输入数据为 lncRNA 表达丰度表数据。对于有生物学重复的样品，分析我们采用基于负二项分布的 DESeq2 进行分析具体使用基于 BiocManager，getopt，ggplot2，DESeq2 等 R包的 R 语言脚本；对各个样本分组进行两两之间的比较，找到在不同分组中表达差异的 lncRNA。lncRNA 表达差异分析结果见下表。

1. _ID:基因/lncRNA名称。_
2. _baseMean：样本标准化序列数的均值。_
3. _log2FoldChange: 处理组与对照组基因表达水平的比值，再经过差异分析软件收缩模型处理，最后以 2 为底取对数。_
4. _p.value：显著性 p 值。_
5. _padj：p 值经过多重校验校正后的值_

我们对差异分析结果绘制火山图，横坐标代表 lncRNA 在不同实验组中/不同样品中表达倍数变化，纵坐标代表 lncRNA 表达量变化的统计学显著程度，图中的散点代表各个 lncRNA 。蓝色和橙色为显著下调和显著上调的基因，默认显著性的判定标准为(p.adj小于0.05，|log2FoldChange|大于1)。

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284735006-1b4f69f4-c907-4f89-bc0c-6bba107c6eba.png)

图3.6.1 表达差异分析火山图

X 轴表达量， Y 轴表示差异倍数，理论上绝大部分 lncRNA 是不显著差异，所以 FC 峰值位置应位于 0 附近，点呈正态分布并对称分布。

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284735400-1c0bd40a-355f-44cb-ba0e-928a9bcfa3a0.png)

### 3.7 lncRNA 靶基因预测

lncRNA 靶基因预测分析步骤结果文件夹结构:

--[比较条件]

----Ctrl_vs_Tumor_pred_cis.txt [lncRNA顺式靶基因预测结果]

----Ctrl_vs_Tumor_pred_trans.txt [lncRNA反式靶基因预测结果]

本项目lncRNA 靶基因预测见结果文件[lncRNApipeline_target](https://bioincloud.tech/cloudir/reports/an_lncRNA/lncRNApipeline_target/)

lncRNA 功能主要通过 cis 或 trans(进行 trans 作用分析需要至少 5 个样品)方式作用于蛋白编码靶基因来实现，因此分成两种情况预测 lncRNA 的靶基因

从生物信息学的角度来预测长链非编码 RNA 调控的顺式靶标基因，可以根据由于 lncRNA 在基因组上邻近靶标编码蛋白质基 因直接起调控作用，称为顺式调控( cis-acting regulatory )。根据长链非编码RNA 与已知编码蛋白质基因的距离域值设定为 lncRNA 的上下游 100kb。

根据长链非编码 RNA 与靶标蛋白质基因的位置上较远间接起调控作用，称为反式调控(trans-acting regulatory)。从生物信息学的角度预测长链非编码RNA 调控的反式靶标基因，基于表达量相关性的共表达分析方法，采用皮尔逊相关系数(Pearson correlationcoefficient)且满足 Pearson_coefficient 大于0.95以及P_value小于 0.05 来预测差异 lncRNA 与所有编码蛋白质 mRNA 的关系对，此分析针对大于等于 5 个的数据集。

下表为顺式调控靶基因预测结果。

1. _nc_tid：lncRNA转录本编号。_
2. _nc_gid：lncRNA转录本所属基因编号。_
3. _cis_gid: 靶基因编号_
4. _cis_name：靶基因名称。_

### 3.8 差异 lncRNA 靶基因功能富集分析

富集分析步骤结果文件夹结构:

--all/up/down

----go_bp/cc/mf_dot_plot.pdf/svg [lncRNA靶基因富集分析点图]

----go_bp/cc/mf_bar_plot.pdf/svg [lncRNA靶基因富集分析柱状图]

----go_bp/cc/mf_results.csv [lncRNA靶基因富集分析结果]

----kegg_bar_plot.pdf/svg [lncRNA靶基因通路分析柱状图]

----kegg_dot_plot.pdf/svg [lncRNA靶基因富集分析点图]

----pathway_results.csv [lncRNA靶基因通路分析结果]

----[kegg通路].png [lncRNA靶基因通路图]

本项目富集分析结果见结果文件[lncRNApipeline_enrich_analysis](https://bioincloud.tech/cloudir/reports/an_lncRNA/lncRNApipeline_enrich_analysis/)

#### 3.8.1 lncRNA靶基因GO与KEGG富集分析

我们对差异靶基因进行富集分析。通过富集分析，可以找到不同条件下的差异靶基因与哪些生物学功能或通路显著性相关，从而揭示和理解生物学过程的基本分子机制。

我们采用clusterProfiler软件对差异靶基因集进行GO功能富集分析，KEGG通路富集分析等。富集分析基于超几何分布原理，其中差异靶基因集为miRNA差异分析所得对应靶基因并注释到GO或KEGG数据库的基因集，背景基因集为所有参与差异分析并注释到GO或KEGG数据库的基因集。富集分析结果是对每个差异比较组合的所有差异基因集进行富集。

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284734189-2ab8c673-f44e-407d-98d6-3dbefcb76994.png)

图3.8.1 GO富集分析原理图

GO(Gene Ontology)是描述基因功能的综合性数据库（http://www.geneontology.org/)，可分为生物过程（Biological Process）和细胞组成（Cellular Component）分子功能（Molecular Function）三个部分。GO 功能显著性富集分析给出与基因集背景相比，在差异表达基因中显著富集的 GO 功能条目，从而给出差异表达基因与哪些生物学功能显著相关。该分析首先把所有差异表达基因向 Gene Ontology 数据库的各个 term 映射，计算每个 term 的基因数目，然后找出与整个基因集背景相比，在差异表达基因中显著富集。GO富集分析方法为clusterProfiler的enricher函数，该方法使用超几何分布检验的方法获得显著富集的GO Term。统计被显著富集的各个GOterm中的基因数，GO富集以padj小于0.05为显著富集。

1. _ID: 富集到的条目(如KEGG通路)的ID_
2. _Description: 富集到的条目(如KEGG通路)的描述_
3. _GeneRatio: 匹配到该条目的差异基因数/差异基因总数_
4. _BgRatio: 该条目一共有多少基因/所有条目一共有多少基因_
5. _pvalue: 显著性检验p值_
6. _p.adjust: BH方法校正后的p值_
7. _qvalue: 校正错误发现率(FDR)的p值_
8. _geneID: 与该条目相匹配的差异基因ID或基因名_
9. _Count: 匹配到该条目的差异基因数_

根据GO富集分析结果，在GO的三个分类中（cellular_component:细胞组分；biological_process：生物学过程；molecular_function：分子功能）分别统计每个分类上调基因、下调基因及所有基因的数目，以柱状图展示。

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284734341-ee688da5-a3d9-4d74-aeb7-825e1c99399b.png)

图3.8.2 靶基因GO富集柱状图

在生物体内，不同基因相互协调行使其生物学功能，通过Pathway显著性富集能确定差异表达基因参与的最主要生化代谢途径和信号转导途径。

KEGG(Kyoto Encyclopedia of Genes and Genomes)是有关Pathway的主要公共数据库，其中整合了基因组化学和系统功能信息等众多内容。信息分析时，我们将KEGG数据库按照动物，植物，真菌等进行分类，依据研究物种的种属选择相应的类别进行分析。Pathway显著性富集分析以KEGG Pathway为单位，应用超几何检验，找出差异基因相对于所有有注释的基因显著富集的pathway，KEGG通路富集同样以padj小于0.05作为显著性富集的阈值。

差异靶基因KEGG富集气泡图是KEGG富集分析结果的图形化展示方式。在此图中，KEGG富集程度通过GeneRatio、p.adjust和富集到此通路上的基因个数来衡量。其中Count指差异表达的基因中位于该pathway条目的基因数目。p.adjust是做过多重假设检验校正之后的Pvalue，越接近于零，表示富集越显著。我们挑选了富集最显著的20条pathway条目在该图中进行展示，若富集的pathway条目不足20条，则全部展示。

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284734575-1aa27a7f-4a06-43e4-8dae-5a99f2a89da6.png)

图3.8.3 KEGG通路富集气泡图

### 3.9 可变剪接分析

可变剪接分析步骤结果文件夹结构:

--[比较条件]

----[可变剪接类型].MATS.JC.txt [可变剪接分析结果]

----[可变剪接类型].MATS.JCEC.txt [可变剪接分析结果]

----summary.txt [可变剪接统计表]

----[可变剪接类型]_Sashimi_plot [可变剪接Sashimi图]

------[可变剪接名称].pdf

本项目可变剪接分析结果见结果文件[lncRNApipeline_Altersplicing](https://bioincloud.tech/cloudir/reports/an_lncRNA/lncRNApipeline_Altersplicing/)

#### 3.9.1 可变剪接类型

我们利用lncRNA-Seq数据进行可变剪接分析，可变剪接（ Alternative Splicing ，AS ）是大多数真核生物细胞中普遍存在的转录本形式。真核生物细胞的基因序列包含内含子（ intron ）和外显子（ exon ），在基因转录成前体 RNA 后内含子会被剪接体移除，而外显子连接进一步加工后成为成熟的 mRNA或lncRNA。

转录出来的前体RNA可以有多种外显子剪接形式，因此使得一个基因在不同时间、不同环境中可以翻译出不同的蛋白质，增加其生理状态下系统的复杂性和适应性。

rMATs是一款可变剪接分析软件，它不仅可以对可变剪接事件进行分类，还可以进行不同样本间可变剪接事件的差异分析。rMATs可鉴定的可变剪接事件分类如下。

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284734933-b3712b17-185a-424a-a8c0-d21e835ea23c.png)

图3.9.1 rMATs可鉴定的可变剪接事件

1. _SE: Skipped exon 外显子跳跃_
2. _MXE: Mutually exclusive exon 外显子互斥_
3. _A5SS: Alternative 5' splice site 5’端外显子发生可变剪接_
4. _A3SS: Alternative 3' splice site 3’端外显子发生可变剪接_
5. _RI: Retained intron 内含子滞留_

#### 3.9.2 差异可变剪接事件

以每个进行差异可变剪接分析的比较组为单位，我们首先统计发生的可变剪切事件的种类及数量，然后分别计算每类可变剪切事件表达量，最后对每类可变剪切事件进行差异分析。我们使用rMATS 软件进行AS 事件的定量及差异分析(参数：-t paired --readLength 100 --variable-read-length --cstat 0.0001)。

各样本的可变剪接事件统计如下表

1. _EventType: 可变剪接类型_
2. _EventTypeDescription: 可变剪接类型全称_
3. _TotalEventsJC: 可变剪接总数目（JC统计方法）_
4. _TotalEventsJCEC: 可变剪接总数目（JCEC统计方法）_
5. _SignificantEventsJC: 显著差异可变剪接数目（JC统计方法）_
6. _SigEventsJCSample1HigherInclusion: 在差异比较组合中处理组高表达的可变剪接事件数目（JC统计方法）_
7. _SigEventsJCSample2HigherInclusion: 在差异比较组合中对照组高表达的可变剪接事件数目（JC统计方法）_
8. _SignificantEventsJCEC: 显著差异可变剪接数目（JCEC统计方法）_
9. _SigEventsJCECSample1HigherInclusion: 在差异比较组合中处理组高表达的可变剪接事件数目（JCEC统计方法）_
10. _SigEventsJCECSample2HigherInclusion: 在差异比较组合中对照组高表达的可变剪接事件数目（JCEC统计方法）_

rMATS 软件可以将 AS 事件进行5 种分类，并且可以对有生物学重复的样品进行差异 AS 分析。每个可变剪接事件对应两个 Isoform，分别为 Exon InclusionIsoform 和 Exon Skipping Isoform，分别对两个 Isoform 进行表达量统计，并除以其有效长度，得到校正后的表达量，然后计算 Exon Inclusion Isoform 在两个Isoform 总表达量的比值，比值即为结果文件中的 IncLevel1 (处理组)和 IncLevel2(对照组)，最后进行差异显著性分析 。

1. _ID: 可变剪接ID_
2. _GeneID: 可变剪接事件所在基因编号_
3. _geneSymbol: 可变剪接事件所在基因名称_
4. _chr: 可变剪接事件所在染色体_
5. _strand: 可变剪接事件所在染色体链的方向_
6. _exonStart_0base: 可变剪接事件跳跃外显子起始位置，以 0 开始计数_
7. _exonEnd: 可变剪接事件跳跃外显子终止位置_
8. _upstreamES: 可变剪接事件跳跃外显子的上游 exon 起始位置_
9. _upstreamEE: 可变剪接事件跳跃外显子的上游 exon 终止位置_
10. _downstreamES: 可变剪接事件跳跃外显子的下游 exon 起始位置_
11. _downstreamEE: 可变剪接事件跳跃外显子的下游 exon 终止位置_
12. _ID.1: 与ID一样_
13. _IJC_SAMPLE_1: 可变剪接事件 Exon Inclusion Isoform 在差异比较组合中处理组的表达量_
14. _SJC_SAMPLE_1: 可变剪接事件 Exon Skipping Isoform 在差异比较组合中处理组的表达量_
15. _IJC_SAMPLE_2: 可变剪接事件 Exon Inclusion Isoform 在差异比较组合中对照组的表达量_
16. _SJC_SAMPLE_2: 可变剪接事件 Exon Skipping Isoform 在差异比较组合中对照组的表达量_
17. _IncFormLen: 可变剪接事件 Exon Inclusion Isoform 的有效长度_
18. _SkipFormLen: 可变剪接事件 Exon Skipping Isoform 的有效长度_
19. _PValue: 可变剪接事件表达差异显著性 p 值_
20. _FDR: 可变剪接事件表达差异显著性 FDR 值_
21. _IncLevel1: 处理组可变剪接事件 Exon Inclusion Isoform 在两个 Isoform 总表达量的比值_
22. _IncLevel2: 对照组可变剪接事件 Exon Inclusion Isoform 在两个 Isoform 总表达量的比值_
23. _IncLevelDifference: IncLevel1 与 IncLevel2 的差值_

对于表达差异显著性的可变剪接事件， 我们用rmats2sashimiplot软件进行可视化展示，SE 类型可变剪接事件的可视化展示如下图所示:图中标题为可变剪接事件三个外显子所在的染色体坐标及正负链信息，跨外显子比对的 reads 使用连接外显子junction 边界的弧线表示。弧线的粗细和比对到 junction 上的 reads 数成正比，同时弧线上的数字指出了 junction reads 的数目，右上方标注了各个样本可变剪接事件所在的基因及 IncLevel 值，横坐标为 AS 发生在染色体上的位置，纵坐标表示发生该可变剪切的 reads 数。图中红色和橘黄色分别表示不同的样本，inclusion level 值代表 exon Inclusion Isoform 在两个 Isoform总表达量的比值，可以直接看出不同样本中可变剪接事件的表达差异。

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284734961-402b2461-8c41-4f81-9ba4-5c7bc9dffbc2.png)

图3.9.1 SE 类型可变剪接可视化展示(Sashimi_plot)

### 3.10 变异位点分析

变异位点分析步骤结果文件夹结构:

--[分组]_allmiRNA_target.txt

--[分组]_allmiRNA_target_up.txt

--[分组]_allmiRNA_target_down.txt

--mutation_type_indel.png [Indel突变类型柱状图]

--mutation_type_snp.png [Snp突变类型柱状图]

--merged_indel_anno.vcf [Indel突变注释文件]

--merged_snp_anno.vcf [Snp突变注释文件]

--merged.variant.vcf [鉴定出的突变文件]

本项目变异位点分析结果见结果文件[miRNApipeline_target_predict](https://bioincloud.tech/cloudir/reports/an_lncRNA/miRNApipeline_target_predict/)

#### 3.10.1 变异位点分析结果

这里分析的变异位点主要有两种类型: SNP和Indel。SNP全称Single Nucleotide Polmorphisms，单核苷酸多态性。是指在基因组上由单个核苷酸变异形成的遗传标记，其数量巨大，多态性丰富。一般而言，SNP是指变异频率大于1%的单核苷酸变异。

Indel全称Insertion deletion，小片段插入缺失。是指相对于参考基因组，样本中发生的小片段的插入缺失，该插入缺失可能含一个或者多个碱基。

变异位点分析是RNA-Seq结构分析的重要内容，主要包括先天变异位点和后天体细胞突变的检测，对肿瘤等研究具有重要意义。我们使用 GATK 软件对每个样本的比对数据(bam)进行变异位点分析(关键参数: gatk HaplotypeCaller --emit-ref-confidence GVCF --dont-use-soft-clipped-bases --standard-min-confidence-threshold-for-calling 30，详细过程参照官网)，并使用VEP(variant effect predictor)软件对变异位点进行注释(参数: --canonical --symbol --vcf --stats_text --gtf [参考基因组注释文件])。

merged_*_anno.vcf中保存了获取变异位点的过程的相关信息及变异位点的具体信息。##开头的行用于对表格内容进行注释，帮助理解表格内容，#开头的行是表头，之后的行即为表格记录，##开头的可能有几百行，需要往下拉很多才能看到表格内容。可用Excel打开，因表格过大，此处仅以示例图片展示，具体以文件内容为准。

表3.10.1 vcf格式变异位点信息及其注释信息

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284735572-ea78ad2d-3ba5-4e1d-a7ac-6408c06d3a34.png)

1. _CHROM: 变异所在染色体编号_
2. _POS: 突变位点在染色体上的坐标_
3. _ID: 突变ID（空）_
4. _REF: 参考基因组的该位点的碱基_
5. _ALT:样本中检测到的变异碱基(对Indel来说是片段)，即等位基因型，可有多个，逗号隔开_
6. _QUAL: 变异位点质量，Phred 格式的数值，数值越高，变异位点基因型的可信度越高_
7. _FILTER: 筛选规则（空）_
8. _INFO:位点注释信息_
9. _FORMAT: ”:“分隔的字段名，对应后续各样本列的字段值_
10. _Sample*:”:“分隔的字段值，对应FORMAT列的字段名；各字段的含义可以在文件开头##开头的行里找到；DP:变异位点在样本中的测序深度；AD:分别指支持REF 与 ALT 的 reads 数，中间以逗号相隔；GT:变异位点的基因型，/分隔，二倍体有两个值，0，1，2，3....分别对应REF以及ALT里逗号分隔的多个等位基因型，也就是说，0代表没有变异_

我们在结果文件夹中提供了按照染色体，变异后果类型，变异严重程度分类的变异基因型的计数总和统计柱形图，下图展示了按照变异严重程度分类的变异基因型的计数统计柱形图。横轴为不同的样本，红色为高严重程度，绿色和蓝色为中等和低严重程度。

![](https://cdn.nlark.com/yuque/0/2024/png/975582/1728284735782-7b64d0d7-43b7-4e20-9c71-fcb0c760df66.png)

图3.10.1 按变异严重程度分类的变异位点计数统计图

## 4 参考文献

- [1] Shifu Chen, Yanqing Zhou, Yaru Chen, Jia Gu; fastp: an ultra-fast all-in-one FASTQ preprocessor, Bioinformatics, Volume 34, Issue 17, 1 September 2018, Pages i884–i890, https://doi.org/10.1093/bioinformatics/bty560IF: 4.4 Q1
- [2] Kim D, Langmead B and Salzberg SL. HISAT: a fast spliced aligner with low memory requirements. Nature Methods 2015
- [3] Okonechnikov, K. , Conesa, A. , & F García-Alcalde. (2015). Qualimap v2: advanced quality control ofhigh throughput sequencing data. Bioinformatics, btv566.
- [4] Pertea et al. (2015) StringTie enables improved reconstruction of a transcriptome from RNA-seq reads.
- [5] Pertea G and Pertea M. "GFF Utilities: GffRead and GffCompare". F1000Research 2020, 9:304 DOI: 10.12688/f1000research.23297.1
- [6] Jaime, H. C. , Kristoffer, F. , Pedro, C. L. , Damian, S. , Juhl, J. L. , & Christian, V. M. , et al. (2016). Fast genome-wide functional annotation through orthology assignment by eggnog-mapper. Molecular Biology & Evolution(8), 2115.
- [7] Mckenna, A. , Hanna, M. , Banks, E. , Sivachenko, A. , Cibulskis, K. , & Kernytsky, A. , et al. (2010). The genome analysis toolkit: a mapreduce framework for analyzing next-generation dna sequencing data. Genome Research, 20(9), 1297-1303.
- [8] Mclaren, W. , Gil, L. , Hunt, S. E. , Riat, H. S. , Ritchie, G. , & Thormann, A. , et al. (2016). The ensembl variant effect predictor. Genome Biology, 17(1), 122.
- [9] Shen, S. , Park, J. W. , Lu, Z. , Lin, L. , MD Henry, & Wu, Y. N. , et al. (2014). rMATS: robust and flexible detection of differential alternative splicing from replicate rna-seq data. Proc Natl Acad Sci U S A, 111(51), 5593-601.
- [10] https://github.com/ddpinto/rmats2sashimiplot
- [11] Yang, L. , Smyth, G. K. , & Wei, S. . (2014). featureCounts: an efficient general purpose program for assigning sequence reads to genomic features. Bioinformatics(7), 923-30.
- [12] Love, M. I. , Huber, W. , & Anders, S. . (2014). Moderated estimation of fold change and dispersion for rna-seq data with deseq2. Genome Biology, 15(12), 550.
- [13] Smyth, G. K. . (2010). edgeR: a bioconductor package for differential expression analysis of digital gene expression data. Bioinformatics, 26(1), 139.
- [14] Yu, G. , Wang, L. G. , Han, Y. , & He, Q. Y. . (2012). Clusterprofiler: an r package for comparing biological themes among gene clusters. Omics-a Journal of Integrative Biology, 16(5), 284-287.
- [15] Damian, S. , Andrea, F. , Stefan, W. , Kristoffer, F. , Davide, H. , & Jaime, H. C. , et al. (2015). String v10: protein–protein interaction networks, integrated over the tree of life. Nucleic Acids Research, 43(D1).
- [16] Langfelder P, Horvath S (2008). “WGCNA: an R package for weighted correlation network analysis.” BMC Bioinformatics, 559. https://bmcbioinformatics.biomedcentral.com/articles/10.1186/1471-2105-9-559IF: 2.9 Q1 .

  
  

## 5 分析所用软件版本

|   |   |
|---|---|
|软件|版本|
|fastp|0.23.1|
|fastqc|0.12.0|
|multiqc|v1.17|
|HISAT2|2.2.1|
|PLEK|v1.2|
|CPC2|none|
|CPAT|3.0.5|
|StringTie|2.1.5|
|gffcompare|v0.12.6|
|emapper|2.0.1|
|GATK|v4.2.2.0|
|rMATS|v4.1.1|
|rmats2sashimiplot|none|
|featureCounts|v2.0.1|
|DESeq2|1.26.0|
|edgeR|3.28.1|
|clusterProfiler|3.14.3|