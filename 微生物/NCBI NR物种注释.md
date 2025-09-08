### **一、数据准备**

#### 1. **下载必要文件**

|                       |                                                                     |                |                                  |
| --------------------- | ------------------------------------------------------------------- | -------------- | -------------------------------- |
| 文件类型                  |                                                                     | 版本             | 作用                               |
| **NR数据库**             | **nr.gz**                                                           | **2024-02-07** | 原始蛋白序列库                          |
| **分类数据库**             | **taxdump.tar.gz**                                                  | **2025-06-12** | 含nodes.dmp（分类层级）、names.dmp（物种名称） |
| **Accession-TaxID映射** | **prot.accession2taxid.FULL.gz**<br><br>**prot.accession2taxid.gz** | **2025-06-06** | 蛋白序列与TaxID对应关系<br><br>完整版和简化版    |

|                       |                                      |                |                                  |
| --------------------- | ------------------------------------ | -------------- | -------------------------------- |
| 文件类型                  |                                      | 版本             | 作用                               |
| **NR数据库**             | **nr.gz**                            | **2024-02-07** | 原始蛋白序列库                          |
| **分类数据库**             | **taxdump.tar.gz**                   | **2025-08-26** | 含nodes.dmp（分类层级）、names.dmp（物种名称） |
| **Accession-TaxID映射** | **prot.accession2taxid.FULL.gz**<br> | **2025-08-22** | 蛋白序列与TaxID对应关系                   |
|                       |                                      |                |                                  |
microNR
```shell
# bacteria, archaea,virus, fungi
2,2157,10239,4751

mkdir microNR
taxonkit list --ids 2,2157,4751,10239 --indent "" > microNR.taxid

pigz -dc prot.accession2taxid.gz \
| csvtk grep -t -f taxid -P microNR.taxid \
| csvtk cut -t -f accession.version,taxid \
| sed 1d > microNR.acc2taxid


cat <(echo) <(pigz -dc nr.gz) \
| perl -e 'BEGIN{ $/ = "\n>"; <>; } while(<>){s/>$//; $i = index $_, "\n"; $h = substr $_, 0, $i; $s = substr $_, $i+1; if ($h !~ />/) { print ">$_"; next; }; $h = ">$h"; while($h =~ />([^ ]+ .+?) ?(?=>|$)/g){ $h1 = $1; $h1 =~ s/^\W+//; print ">$h1\n$s";} } ' \
| seqkit grep -f microNR.taxidt -o microNR.fa.gz

cut -f 1 $id.acc2taxid.txt > $id.acc.txt
```
https://scienceparkstudygroup.github.io/ibed-bioinformatics-page/source/core_tools/ncbi_nr.html
https://bioinf.shenwei.me/taxonkit/tutorial/

💡 **验证完整性**：使用`md5sum -c *.md5`校验文件。

```
https://ftp.ncbi.nih.gov/blast/db/FASTA/nr.gz
https://ftp.ncbi.nih.gov/blast/db/FASTA/nr.gz.md5

https://ftp.ncbi.nih.gov/pub/taxonomy/taxdump.tar.gz
https://ftp.ncbi.nih.gov/pub/taxonomy/taxdump.tar.gz.md5

https://ftp.ncbi.nih.gov/pub/taxonomy/accession2taxid/prot.accession2taxid.FULL.gz
https://ftp.ncbi.nih.gov/pub/taxonomy/accession2taxid/prot.accession2taxid.FULL.gz.md5

https://ftp.ncbi.nih.gov/pub/taxonomy/accession2taxid/prot.accession2taxid.gz
https://ftp.ncbi.nih.gov/pub/taxonomy/accession2taxid/prot.accession2taxid.gz.md5
```

diamond 数据库

```
mkdir taxdmp && tar -xf taxdump.tar.gz -C taxdump
diamond makedb \
--verbose \
--log \
--threads 47 \
--in nr.gz \
--db NR \
--taxonmap prot.accession2taxid.FULL.gz \
--taxonnodes taxdump/nodes.dmp \
--taxonnames taxdump/names.dmp
```

diamond + taxonkit

diamond 对比

```
diamond blastp \
--db NR \
--taxonlist 2,2157,10239 \
--query protein.faa \
--threads 47 \
--log \
-–range-culling -k 25 -F 15 \
--tmpdir tmp \
--evalue 1e-03 \
--top 10 \
--sensitive \
--outfmt 6 qseqid sseqid pident length mismatch gapopen qstart qend sstart send evalue bitscore staxids sscinames salltitles \
--include-lineage \
--out diamond
```

1. `--range-culling`  
    这是 Diamond 中用于优化比对结果的筛选参数，核心作用是**去除同一查询序列与同一目标序列之间的重叠匹配**。当一个查询蛋白与同一个目标蛋白存在多个局部比对结果（例如因序列分段匹配或重复结构域导致），该参数会保留得分最高的非重叠区域，剔除其他重叠的冗余匹配。这有助于减少结果中的冗余信息，确保每个 “查询 - 目标” 对仅保留最有价值的匹配区域，尤其适用于分析含重复结构域的蛋白质序列。
    
2. `-k 25`  
    用于指定**每个查询序列保留的最大匹配结果数量**，此处设置为 25，即对输入文件中的每个蛋白序列，仅保留得分最高的前 25 个目标序列匹配结果。该参数平衡了结果的全面性与计算效率：值过大会导致结果冗余（尤其是高度保守的蛋白可能匹配上百个同源序列），值过小可能遗漏重要同源信息。25 是中间值，适用于多数需兼顾敏感性和简洁性的分析场景（如功能注释初筛）。
    
3. `-F 15`  
    该参数控制序列比对前的**低复杂度区域过滤**，数值 “15” 对应 Diamond 的内置过滤规则组合，具体等效于：
    
    - 启用对查询序列和目标序列中低复杂度区域（如富含某种氨基酸的重复序列）的屏蔽（类似 BLAST 中的`-seg`或`-dust`参数）；
    - 这些区域通常因序列组成特殊而容易产生假阳性匹配（非生物学意义的相似性），过滤后可提高比对结果的可靠性。  
        若设置为`-F 0`则关闭过滤，适用于需要保留低复杂度区域（如研究特定重复序列功能）的场景，但可能增加假阳性。

```
taxonkit reformat2 \
--taxid-field 13 \
--format "d__{domain|acellular root|superkingdom}\tp__{phylum}\tc__{class}\to__{order}\tf__{family}\tg__{genus}\ts__{species}" \
--data-dir microNR/taxdump \
--no-ranks \
--fill-miss-rank \
--threads 45 \
--out-file taxonkit.tsv \
diamond
```

得到taxonkit.tsv 后一个基因id对应多个物种，可以后续考虑设计一个打分方法根据分数选择一个最优的注释，
1. 首先选择注释等级最精细删除注释不那么精细的
2. 如果这时注释等级相同，那么删除带有uncultured的注释
3. 从doamin开始往后对比，找到开始不一样的等级，统计此等级注释分别的频率，多数服从少数，保留多数注释
4. 按照3的方法继续往后筛选，直到仅有一条注释位置
5. 如果此时仍然有多余一条注释，那么选择他们等级相同的注释为最终注释（比如仅注释到目水平，后边的注释信息因为不同所以舍弃）

```R
Canton::using(data.table)
a=fread('/home/zktbk0/Alex/taxonkit.tsv.gz')
b=data.table::unique(a)
dt <- b[V4 != "p__"]
data.table::setorder(dt, V1)
fwrite(head(dt,1000),'1.ccsv')
```

diamond + MEGAN

```
diamond blastp \
--db NR \
--taxonlist 2,2157,10239 \
--query protein.faa \
--threads 47 \
--log \
-–range-culling -k 25 -F 15 \
--tmpdir tmp \
--evalue 1e-03 \
--top 10 \
--outfmt 100 \
--sensitive \
--include-lineage \
--out diamond


diamond blastp \
--db /data/Alex/NR.dmnd \
--taxonlist 2,2157,10239 \
--query 03.X207.faa.rmdup.faa \
--threads 31 \
-b 0.5 \
--log \
--evalue 1e-05 \
--top 10 \
--outfmt 100 \
--sensitive \
--include-lineage \
--out diamond
```

```

wget https://software-ab.cs.uni-tuebingen.de/download/megan6/megan-map-Feb2022.db.zip

megan=/home/data/wd/Vik/APP/megan/tools
# diamond dda ->  megan dda
$megan/daa-meganizer \
--longReads \
--in /home/data/wd/Vik/X207/Result/X207.dda.daa \
--mapDB /home/data/wd/Vik/X207/DB/megan-map-Feb2022.db \
-lcaAlgorithm longReads \
--threads 55 --classify true

# megan dda -> megan rma
$megan/daa2rma \
--in /home/data/wd/Vik/X207/Result/X207.dda.daa \
--out /home/data/wd/Vik/X207/Result/X207.rma \
--mapDB /home/data/wd/Vik/X207/DB/megan-map-Feb2022.db \
--longReads --lcaAlgorithm longReads \
--threads 55 --classify true \
--minScore 50 --maxExpected 0.01 --topPercent 50 

# mapDB: MEGAN映射库路径
# minScore: 最小支持数
# maxExpected: 最大期望值
# topPercent: 保留Top N比对结果
# longReads: 对长序列（如contigs）启用优化

# megan rma -> table
$megan/rma2info \
--in /home/data/wd/Vik/X207/Result/X207.rma \
--out /home/data/wd/Vik/X207/Result/X207.tsv \
--class2count Taxonomy --read2class Taxonomy \
--ignoreUnassigned true \
--names true --paths true --ranks true --list true --listMore true --majorRanksOnly true --verbose
# read2class: 指定输出分类信息
# names: 显示物种名称
# paths: 输出完整分类路径（界→门→属→种）
# ranks: 包含分类层级标识
# list: 以列表格式输出
# listMore: 包含更多信息
# majorRanksOnly: 仅保留主要分类层级（界/门/纲/目/科/属/种）
# ignoreUnassigned: 忽略未分类的序列
```

## Reference

- [https://software-ab.cs.uni-tuebingen.de/download/megan6/welcome.html](https://software-ab.cs.uni-tuebingen.de/download/megan6/welcome.html)
- [https://github.com/husonlab/megan-ce](https://github.com/husonlab/megan-ce)
- [https://software-ab.cs.uni-tuebingen.de/download/megan6/manual.pdf](https://software-ab.cs.uni-tuebingen.de/download/megan6/manual.pdf)
- [https://blog.csdn.net/woodcorpse/article/details/104381643](https://blog.csdn.net/woodcorpse/article/details/104381643)
- https://www.jianshu.com/p/1d6edfcb4110


一、前列腺癌甲基化预后标志物筛选系统​

该系统聚焦前列腺癌甲基化数据的深度挖掘与预后标志物筛选，具备完整的 “数据获取 - 处理 - 分析 - 成果输出” 闭环功能。首先，通过自动化接口精准下载 TCGA 数据库中前列腺癌样本的甲基化原始数据，同时内置数据质控模块，可自动过滤低质量探针、去除批次效应及样本异常值，确保数据可靠性；接着，采用 limma 算法结合差异倍数与统计显著性阈值，高效筛选出癌组织与正常组织间的差异甲基化位点，并生成可视化火山图与热图，直观呈现位点表达差异特征；随后，针对筛选出的差异位点，创新性整合单因素 Cox 比例风险回归模型与 log-rank 检验两种分析方法，前者用于量化位点与患者生存时间的关联强度（计算 HR 值及 95% 置信区间），后者用于验证不同甲基化水平分组的生存曲线差异，双重验证确保预后标志物的准确性；最后，系统可自动生成包含差异位点信息、Cox 分析结果、生存曲线及统计检验表格的综合报告，为临床前列腺癌预后评估及潜在治疗靶点研究提供直接的数据支撑与可视化依据。​

二、前列腺癌基因突变分析系统​

该系统专为前列腺癌基因突变特征解析与临床关联研究设计，覆盖基因突变数据全流程分析需求。在数据处理环节，系统支持批量导入 TCGA 数据库的前列腺癌基因突变 MAF 格式文件，通过内置的突变注释工具，自动完成基因突变类型（错义突变、无义突变、移码插入 / 缺失等）的分类标注，同时过滤胚系突变与低置信度突变位点，保障分析数据的临床相关性；在突变特征统计方面，系统可自动计算各基因的突变频率、突变频谱（碱基替换模式）及样本突变负荷，生成 oncoplot 瀑布图、突变类型分布饼图等可视化图表，清晰展示高频突变基因的突变模式与样本突变分布规律；此外，系统核心功能在于突变与预后的关联分析，通过 Kaplan-Meier 生存分析结合 log-rank 检验，深入探究高频突变基因（如 TP53、PTEN、SPOP 等）的突变状态与患者总生存期（OS）、无进展生存期（PFS）的关联，同时支持亚组分析，可按临床病理特征（如肿瘤分期、分级）划分样本组，分析不同亚组中突变的预后意义；最后，系统还具备突变基因功能富集分析模块，可对高频突变基因进行 GO 功能注释与 KEGG 通路富集分析，揭示突变基因涉及的生物学过程（如细胞周期调控、DNA 损伤修复），为解析前列腺癌发生发展机制及指导靶向治疗方案选择提供关键技术支持。