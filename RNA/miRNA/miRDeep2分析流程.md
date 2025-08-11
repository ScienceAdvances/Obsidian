[https://www.jianshu.com/p/bb3536fc1744](https://www.jianshu.com/p/bb3536fc1744)

# 准备数据库

```
curl -s http://www.regulatoryrna.org/database/piRNA/download/archive/v2.0/fasta/hsa.fa.gz | zcat - > database/pirna.fasta 2> database/logs/get_pirna_index.log 
curl -s ftp://mirbase.org/pub/mirbase/CURRENT/mature.fa.gz | zcat - | seqkit grep -r -p ^hsa | seqkit seq --rna2dna > database/mirna_mature.fasta 2> database/logs/get_mirna_sequences.log 
curl -s ftp://mirbase.org/pub/mirbase/CURRENT/hairpin.fa.gz | zcat - | seqkit grep -r -p ^hsa | seqkit seq --rna2dna > database/mirna_hairpin.fasta 2> database/logs/get_mirna_sequences.log 
cat database/mirna_mature.fasta database/mirna_hairpin.fasta  > database/mirna.fasta

bowtie-build database/pirna.fasta database/pirna > database/logs/get_pirna_index.log
bowtie-build database/mirna.fasta database/mirna > database/logs/get_mirna_index.log
```

```

bowtie2 --very-sensitive-local \
    -p 1 -x database/mirna \
    --un-gz results/sampleA/fastq/sampleA_mirna_unmapped.fastq.gz \
    results/sampleA/fastq/sampleA_trimmed.fastq.gz | samtools sort - > results/sampleA/bam/sampleA_mirna.bam 2> results/sampleA/logs/map_mirna_sequences.log 

samtools index results/sampleA/bam/sampleA_mirna.bam results/sampleA/bam/sampleA_mirna.bai
```