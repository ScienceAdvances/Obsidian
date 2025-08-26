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
|                       |                                      |                |                                  |
|                       |                                      |                |                                  |
|                       |                                      |                |                                  |
|                       |                                      |                |                                  |
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
--outfmt 6 qseqid sseqid pident length mismatch gapopen qstart qend sstart send evalue bitscore staxids sscinames \
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
taxonkit reformat \
--format "{d}\t{p}\t{c}\t{o}\t{f}\t{g}\t{s}" \
--data-dir taxdump \
--prefix-d "k__" \
--taxid-field 3 \
--fill-miss-rank \
--add-prefix \
--threads 16 \
--out-file taxonkit.tsv \
diamond.txt
```

合并 join

```
import dask.dataframe as dd
df_left = dd.read_csv("/home/data/wd/Vik/X207/Result/alig_result.xls",sep="\t",header=None)
df_right = dd.read_csv("/home/data/wd/Vik/X207/Result/prot.accession2taxid.FULL",sep="\t")
result = df_left.merge(df_right, left_on=1,right_on="accession.version", how="inner")
result.to_csv("/home/data/wd/Vik/X207/tmp/output_*.tsv", index=False, single_file=False,sep="\t",header=False)
cat output_*.tsv > acc_taxid.xls
awk -F'\t' '$3!=""' acc_taxid.xls > acc_taxid_filter.xls

zcat /home/data/wd/Vik/X207/DB/salmon.counts.xls.gz | head -n 3000 | \
awk 'BEGIN {FS=OFS="\t"} 
     NR==FNR {b[$1]=1; next} 
     FNR>1 && ($1 in b) {print $0}' /home/data/wd/Vik/X207/Result/taxnomy.xls - > /home/data/wd/Vik/X207/Result/count.xls

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

[https://software-ab.cs.uni-tuebingen.de/download/megan6/welcome.html](https://software-ab.cs.uni-tuebingen.de/download/megan6/welcome.html)

[https://github.com/husonlab/megan-ce](https://github.com/husonlab/megan-ce)

[https://software-ab.cs.uni-tuebingen.de/download/megan6/manual.pdf](https://software-ab.cs.uni-tuebingen.de/download/megan6/manual.pdf)

[https://blog.csdn.net/woodcorpse/article/details/104381643](https://blog.csdn.net/woodcorpse/article/details/104381643)

https://www.jianshu.com/p/1d6edfcb4110