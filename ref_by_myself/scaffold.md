# 20261031在拿到HIC Isoseq illumina数据之后，对yang-v2进行HIC数据的挂载以及illumina数据的回帖提升基因组质量
# 需要说明的是yang-v2就是我自己拼装出来的基因组，yang-v1是T2T+我拼出的线粒体序列
## 拿到数据之后，先对数据进行了一个简单的检查
### 通过查看免疫刺激的序列后30个碱基发现没有去除ploA尾，但是发育组去掉了，所以后续处理去掉了polyA
```
# 查看序列的前30和后30bp
f=isoseq/immune_stimulation/imm_LPS_Int.flnc.fastq.gz
echo "== 3' 端最后 30 bp"; seqkit head -n 15 $f | seqkit subseq -r -30:-1 | seqkit seq -s
echo "== 5' 端前 30 bp";   seqkit head -n 15 $f | seqkit subseq -r 1:30   | seqkit seq -s

# 去除polya
for f in isoseq/immune_stimulation/*.flnc.fastq.gz; do
    name=$(basename "$f" .flnc.fastq.gz)       # 例如 imm_LPS_Int
    echo "== $name"
    cutadapt --poly-a -j 4 \
        -o "${name}.flnc.trimA.fastq.gz" \
        "$f" > "${name}.cutadapt.log"
done
```
### 通过查看hic数据的header发现两批数据是同一个index，有一批数据量少的是补测的，所以后续合并了，又发现了HIC的酶切位点为 GATC
```
for f in hic/*_1.fq.gz; do echo "== $f"; zcat $f | head -1; done

# 取 L2 R1 前 100 万条序列存为临时文件（约 150 MB）
zcat hic/FDHC230007956-1a_L2_1.fq.gz | head -4000000 | awk 'NR%4==2' > tmp_hic_1M.seq

# 统计各种酶的连接序列出现频率
for p in GATCGATC 'GA.TGATC' 'GATCA.TC' 'GA.TA.TC' AAGCTAGCTT CATGCATG; do
  n=$(grep -cE "$p" tmp_hic_1M.seq)
  echo -e "$p\t$n\t$(echo "scale=2; $n/10000" | bc)%"
done

# 合并HIC数据，把两批R1合并到一起，R2合并到一起
cat hic/FDHC230007956-1a_L1_1.fq.gz hic/FDHC230007956-1a_L2_1.fq.gz > hic_merged_R1.fq.gz
cat hic/FDHC230007956-1a_L1_2.fq.gz hic/FDHC230007956-1a_L2_2.fq.gz > hic_merged_R2.fq.gz
```

## 然后对数据进行质检
### Hi-C 质检要先把 read 比对到组装上，所以给 yang-v2 建 bwa 索引
```
samtools faidx yang-v2.fa
bwa index yang-v2.fa
```
```
# 用 L2 原文件抽样，不要从合并文件开头取：合并文件的前 830 万对来自 L1，而 L2 才是 98% 数据所在的正式测序，抽样应该代表主体数据
# 不要对 Hi-C 做剪切：bwa mem -5SP 会自己处理 read 末端，提前剪切反而可能破坏配对信息，这是 YaHS、Arima 等官方流程的通用建议
zcat hic/FDHC230007956-1a_L2_1.fq.gz | head -8000000 > hicqc_R1.fq   # 前 200 万对
zcat hic/FDHC230007956-1a_L2_2.fq.gz | head -8000000 > hicqc_R2.fq
bwa mem -5SP -t 16 yang-v2.fa hicqc_R1.fq hicqc_R2.fq | samtools view -bh - > hicqc.bam
# 然后用 Phase Genomics 的 hic_qc 读这个 BAM 出报告，或者用 pairtools parse → dedup → stats。重点看有效 pair 比例、长程 cis（>10 kb）比例和重复率。
```
### illmina质检
```
# fastp：只看报告，不用输出剪切后的 read。它会给出精确的 read 数、Q30 比例、重复率、接头比例和插入片段大小，正好把表格里的估算值换成实测值
# k-mer 评估：用原始 read 直接建 k-mer 库就行，低质量 k-mer 会落在频数分布的错误峰里，不影响结果
meryl count k=21 threads=16 memory=64 output illumina.k21.meryl illumina/*.fq.gz# 建立一个kmer库
meryl histogram illumina.k21.meryl > illumina.k21.hist     # → GenomeScope2，给kmer画直方图
merqury.sh illumina.k21.meryl yang-v2.fa merqury_yang-v2   # → QV 和完整度，看组装中有什么kmer，哪些是应该有的，哪些是不应该有的
```

## 应该做的工作的顺序：先矫正 contig，再做 Hi-C 挂载。
```
# 矫正会修正小的插入缺失，contig 上每个修正点之后的坐标都会移动几个 bp。
# 而挂载的所有中间结果都绑定在坐标上：Hi-C 比对结果、互作矩阵、YaHS 生成的 AGP 文件、Juicebox 里的手工调整记录。
# 如果先挂载再矫正，这些坐标全部失效，等于挂载要重做一遍，Juicebox 的手工调整也白做了。
```
