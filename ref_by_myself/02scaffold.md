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

# 把 200 万对 Hi-C read 比对到 yang-v2 上，得到每条 read 落在哪个 contig 的哪个位置，再存成按 read 名排序的 BAM 文件
bwa mem -5SP -t 16 yang-v2.fa hicqc_R1.fq hicqc_R2.fq | samtools view -bh - > hicqc.bam
# 然后用 Phase Genomics 的 hic_qc 读这个 BAM 出报告，或者用 pairtools parse → dedup → stats。重点看有效 pair 比例、长程 cis（>10 kb）比例和重复率。
```
### illmina质检
```
# fastp：只看报告，不用输出剪切后的 read。它会给出精确的 read 数、Q30 比例、重复率、接头比例和插入片段大小，正好把表格里的估算值换成实测值
# 切除接头和过滤低质量数据(fastq这个工具有些默认参数会识别低质量的碱基)
fastp -i "$R1" -I "$R2" --detect_adapter_for_pe -w 16 \
  -j fastp/fastp.json -h fastp/fastp.html > fastp/fastp.log 2>&1
# k-mer 评估：用原始 read 直接建 k-mer 库就行，没有用fastq输出的干净的reads，低质量 k-mer 会落在频数分布的错误峰里，不影响结果
meryl count k=21 threads=16 memory=64 output illumina.k21.meryl illumina/*.fq.gz# 建立一个kmer库
meryl histogram illumina.k21.meryl > illumina.k21.hist     # → GenomeScope2，给kmer画直方图
merqury.sh illumina.k21.meryl yang-v2.fa merqury_yang-v2   # → QV 和完整度，看组装中有什么kmer，哪些是应该有的，哪些是不应该有的
```

## 应该做的工作的顺序：先矫正 contig，再做 Hi-C 挂载。
```
# 矫正会修正小的插入缺失，contig 上每个修正点之后的坐标都会移动几个 bp。
# 而挂载的所有中间结果都绑定在坐标上：Hi-C 比对结果、互作矩阵、YaHS 生成的 AGP 文件、Juicebox 里的手工调整记录。
# 如果先挂载再矫正，这些坐标全部失效，等于挂载要重做一遍，Juicebox 的手工调整也白做了。

# 似乎原理是这样的：hifi先比对到组装上，然后在有歧义的地方提供一些候选，比如和组装不一样、单倍体差异这样，会提供几个候选，如果现在的组装的kmer在illumina的kmer中找不到对应（组装错误），那么nextpolisher就会从候选中找一个在illumina的kmer中出现次数多的候选给组装进行修改。如果候选的kmer出现次数一样，也就是真实出现的单倍体差异，那就不修改。
# illumina在重复区、GC含量高的区域可能测不到reads，所以这些地方的QV可能比较低，如果HIFI的reads都一致的话应该就是对的
# 首先做yang-v2的polishing，用到的数据是HIFI Cell 1A和illumina数据，其中hifi数据用于定位错误，比如单倍体冗余的地方，而且因为hifi读长很长可以定位到唯一的repetitive，用illumina来进行修正错误
# 01、切除illumina中的接头和质量差的read，接头不是七鳃鳗的序列。接头会产生错误的 k-mer。错误的 k-mer 会误导矫正。
fastp -i illumina_R1.fq.gz -I illumina_R2.fq.gz \
  -o 01_illumina_clean_R1.fq.gz -O 01_illumina_clean_R2.fq.gz \
  --detect_adapter_for_pe -q 15 -u 40 -n 5 -l 15 -w 16 \
  -j 01_fastp.json -h 01_fastp.html

# 02、把hifi reads比对到yang-v2，先把bam文件转换成fastq文件不输出直接给minimap，然后按照位置排序并建立索引
# 在某个位置，如果大多数 read 和组装不同，这个位置就可能有错误。比对文件提供这些候选位置。
minimap2 -ax map-hifi -t $T yang-v2.fa \
  <(samtools view -u -F 0x900 hifi_1A.bam | samtools fastq -) \
  | samtools sort -@ 8 -m 3G -T 03_sort_tmp -o 03_hifi.sorted.bam -
samtools index 03_hifi.sorted.bam
samtools flagstat 03_hifi.sorted.bam > 03_hifi_flagstat.txt
samtools coverage 03_hifi.sorted.bam > 03_hifi_coverage.tsv

# 03、制作kmer库，统计 Illumina clean reads 中每种 k-mer 的出现次数
# k-mer 库是判断碱基对错的标准。某段序列的 k-mer 在库中出现约 40 次，说明这段序列真实存在。出现次数为 0，说明这段序列可能有错误。
CLEAN="01_illumina_clean_R1.fq.gz 01_illumina_clean_R2.fq.gz"
yak count -t $T -k 21 -b 37 -o 04_k21.yak <(zcat $CLEAN) <(zcat $CLEAN)   # 21-mer 短，统计结果稳定。
yak count -t $T -k 31 -b 37 -o 04_k31.yak <(zcat $CLEAN) <(zcat $CLEAN)   # 31-mer 长，在重复区更容易区分不同位置。-b 37：用约 16 GB 的过滤器去掉只出现一次的 k-mer。这些 k-mer 大多来自测序错误。输入写两次：yak 第一遍读取时填充过滤器，第二遍读取时计数。

# 04、nextpolisher矫正，用比对文件找出组装和 HiFi read 不同的位置，为每个位置列出候选序列，在 k-mer 库中查询每个候选序列的 k-mer，选择 k-mer 支持最多的候选序列。证据不足时，它保留原序列。
nextPolish2 -t $T 03_hifi.sorted.bam yang-v2.fa 04_k21.yak 04_k31.yak \
  > 05_yang-v2.np2.fa 2> 05_nextpolish2.log
samtools faidx 05_yang-v2.np2.fa

# 05、Merqury：比较矫正前后的 QV
# 把矫正后的组装切成 21-mer，建一个组装的 k-mer 库
# 找出"组装里有、Illumina 里没有"的 k-mer，换算成 QV（和质检时的算法一样）
# 找出"Illumina 里可靠、组装里也有"的 k-mer 比例，就是完整度
# 对每条 contig 单独计算一次 QV
merqury.sh /tmpdata/YangWenyan/20261029dalian_data/yang-v3-qc/illumina/meryl/illumina.k21.meryl \
  05_yang-v2.np2.fa 06_merqury_np2# 这几个文件分别是参考标准kmer、校正后的组装、输出文件的前缀

# Illumina 的覆盖度约 44×，按 k-mer 算约 38×。这个数是对两份同源染色体合计而言的，每一份各贡献一半，约 19×。所以，一个 k-mer 在 reads 中出现几次，取决于它在每个细胞的基因组里有几份
# 这个 k-mer 存在于一个位置，只在一套单倍型上（杂合），每个细胞中一份，reads出现约19次，主要的组装中出现1次或0次，组装中随机选了一种，剩下的没被组装进去就是 read-only（只在illumina的reads中出现）
# 这个 k-mer 存在于一个位置，两套单倍型都有（纯合），每个细胞中2份，reads出现约38次，主要的组装中出现1次
# 这个 k-mer 存在于两个不同的位置，两套单倍型都有，每个细胞中4份，reads出现约72次，主要的组装中出现2次，因为主要组装只保留一份单倍型

# 06、bwa index：为 Hi-C 挂载做准备，矫正会改变 contig 上的坐标。挂载的结果都依赖坐标。所以先矫正，后挂载。Hi-C 数据要比对到矫正后的组装上。
bwa index 05_yang-v2.np2.fa
```
## HIC挂载
```
# 检查组装中是否有冗余：即是否同时保留两份单倍体
awk -F'\t' 'NR>1{m=$2;
  r=(m<=5?"1_错误区":m<=28?"2_杂合峰":m<=55?"3_纯合峰":m<=95?"4_两倍纯合峰":"5_更高");
  s[r"\t"$1]+=$3} END{for(k in s) print k"\t"s[k]}'   06_merqury_np2.05_yang-v2.np2.spectra-cn.hist | sort
```
