## 20260917以yang-asm1比对T2T判断breakpoint，并用petmar来做断点仲裁，以此例来深入了解基因组知识
### 基因组中的序列字符串本身永远是 5'→3' 书写的，不确定的是：这一串到底代表染色体的哪条链。所以当yang-asm1中的某一条contig作为query来minimap比对到T2T上时，以query的真实存储链向为+，如果能正好比对到reference上记录链向为+，如果需要query反向互补之后与reference才match则记录链向为-

### 根据基因组和注释从基因组中提取固定序列的CDS和蛋白质序列
```
# gffread中的这些选项没特殊的意义，是随便给的字母而已，格式：gffread -x <输出CDS文件> -g <基因组> <输入GTF>
# -w	转录本序列（外显子拼接，含 UTR）
# -x	CDS 序列（只有编码区的核酸）
# -y	蛋白序列（CDS 翻译成的氨基酸）
# -P：用 Perl 风格的正则表达式来解释搜索内容。提取的文件含这个基因的 gene、transcript、exon、CDS 等所有行。
grep -P 'gene_id "LrY_g027920"' ./yang-v2-annotation.gtf   > ./LrY_g027920.gtf
gffread -x LrY_g027920.CDS.fa -g yang-v2.polish.fa LrY_g027920.gtf
gffread -y LrY_g027920.protein.faa -g yang-v2.polish.fa LrY_g027920.gtf
```

#### 查看bam文件
```
samtools view -h(带着头部信息)/H（只有头部信息）/啥也不写就是看主体部分 xxxx.bam
# 以只看主体部分为例，这样能看到这样几部分内容，首先是这条reads的名字和cell barcode，然后是flag，然后记录比对到哪条contig上以及对应的坐标，比对质量MAPQ，比对方式CIGAR 12S110M（开头 12 个碱基被"软剪切"，即它们没有参与比对，但序列仍保留在记录里，这是试剂盒接头序列。后面 110 个碱基连续比对上了参考序列。），配对read所在的位置，这对 read 所覆盖的片段总长度（碱基数），序列
samtools view xxxx.bam
samtools flags 99# 可以用这条命令拆解99是什么意思，1+2+32+64分别表示：双端测序、正常比对、配对的另一条reads比对到反向链、当前是reads1
```

#### 在进行cellranger的时候保留bam文件，使用bam文件拆分不同个体，以雷氏七鳃鳗LPS PHA处理的血液样本为例，我们使用cellSNP-lite + Vireo的方法进行拆分个体。

#### cellsnp-lite第一步会在bam文件中做每个位置上的碱基统计，如果一个位置同时有相当数量的两种碱基，就记为候选SNP，如果只看到一种碱基，说明这个位置对于区分不同个体没用就不记录在内。第二步只看第一遍选出的候选位点，利用BAM里的CB标签，把reads按细胞分开，输出是两个大矩阵，行是位点，列是细胞，内容是统计每个细胞在每个位点上测到了几次参考碱基、几次变异碱基。
#### vireo根据这张表推断每个细胞属于哪条鱼。先随机猜一个分组。根据当前分组，把同组细胞的计数加起来，估计每条鱼在每个位点上的基因型。根据估计出的基因型，重新计算每个细胞更像哪条鱼，更新分组。重复，直到分组不再变化。例如：两条鱼三个细胞，鱼1是1 2 3细胞，鱼2是4 5 6细胞，第一次根据当前对于每个位点上基因型进行分组，把1 2 4分到了一组， 3 5 6分到了另一组，然后对每个组的细胞的基因型进行统计，比如在第一个位点上都是和参考基因一样的碱基，第二个位点上都是变异的，第三个是2参考 ： 1变异，同理第二组也有一个1 ： 1，然后对每个细胞进行判断，发现3在第三个位点上是参考，于是换到第一组，同理把4换到第二组，这样分组在所有细胞的每一个位点上几乎要么都是参考，要么都是变异占主导，不会出现1 ：1这就是分组正确了。

## 根据bam文件拆分个体
### 01、计算VCF文件
```
#!/bin/bash
# 用途：从 LPS、PHA 两个血液文库生成候选 SNP 位点表（VCF），
# 供 souporcell 或 cellsnp-lite 拆分个体时使用
set -euo pipefail

# ---------- 参数 ----------
export PATH=/home/YangWenyan/miniconda3/envs/souporcell/bin:$PATH
COUNT=/tmpdata/YangWenyan/yang-v2/count_all_v436
OUT=$COUNT/splitsample/common_variants
FASTA=/tmpdata/YangWenyan/yang-v2/refdata-yang-v2.v4.3.6/fasta/genome.fa
SAMPLES=(TF260115LRBLLP TF260115LRBLPH)
THREADS=40
FRAC=0.08        # 每个 BAM 抽取的 reads 比例
SEED=42          # 随机种子，保证可重复

log() { echo "[$(date '+%F %T')] $*"; }
mkdir -p "$OUT"

# ---------- 1. 每个文库抽样（并行） ----------
# 对每个样本做抽样处理，找候选位点只需要知道"哪些位置有两种碱基"，不需要全部数据。保存为bam格式，16线程，抽样比例，随机数种子，输出文件，&意思是放在后台执行
log "抽样，比例 $FRAC"
DS=(); PIDS=()
for S in "${SAMPLES[@]}"; do
  samtools view -b -@ 16 --subsample "$FRAC" --subsample-seed "$SEED" \
    -o "$OUT/$S.ds.bam" "$COUNT/$S/outs/possorted_genome_bam.bam" &
  PIDS+=($!); DS+=("$OUT/$S.ds.bam")# DS记录抽样后的文件名
done
for P in "${PIDS[@]}"; do wait "$P"; done     # 逐个等待，任何一个失败都会让脚本停下

# ---------- 2. 合并并建索引 ----------
# 把两个抽样后的 BAM 合并成一个。合并的目的是把 6 条鱼的 reads 放在一起找变异，得到一张两个文库通用的位点表。
log "合并"
samtools merge -@ "$THREADS" -o "$OUT/pool.ds.bam" "${DS[@]}"
samtools index -@ "$THREADS" "$OUT/pool.ds.bam"# 给合并后的文件建立索引
rm "${DS[@]}"

# ---------- 3. freebayes 找变异 ----------
log "freebayes"
# 把基因组切成每段 100 万碱基的小区域，列表存进 regions.txt，比如：ptg000001l:0-1000000。$FASTA.fai 是基因组的索引文件，里面记着每条 contig 的名字和长度
fasta_generate_regions.py "$FASTA.fai" 1000000 > "$OUT/regions.txt"
# 举例：程序从第 1 个位置走到第 100 万个位置。每到一个位置做同一件事：看参考基因组这里是什么碱基，再看 BAM 里落在这里的 reads 是什么碱基，两者有没有不同。有不同、且符合筛选规则的位置，记下来。
# 参数分别是不要indel、不要多碱基变异、只保留单碱基变异、变异碱基至少有 2 条 reads 支持、只用测序质量20以上的碱基、只用比对质量30以上的 reads，排除能比对到多个位置的、reads 覆盖不到20条的位置直接跳过
freebayes-parallel "$OUT/regions.txt" "$THREADS" -f "$FASTA" \
  -iXu -C 2 -q 20 -m 30 --min-coverage 20 \
  "$OUT/pool.ds.bam" > "$OUT/pool.raw.vcf" 2> "$OUT/freebayes.log"

# ---------- 4. 过滤、排序、建索引 ----------
log "过滤"
# 等位基因数最少 2 个、最多 2 个，也就是只留"一种参考碱基加一种变异碱基"的位点。出现三种碱基的位点多半有问题。只留 SNP。只留质量分高于 30 的。质量分越高，这个位点是真实变异的把握越大。然后按基因组坐标排序
bcftools view -m2 -M2 -v snps -i 'QUAL>30' "$OUT/pool.raw.vcf" \
  | bcftools sort -O z -o "$OUT/common_variants.vcf.gz"
tabix -p vcf "$OUT/common_variants.vcf.gz"

log "候选 SNP 数：$(bcftools view -H "$OUT/common_variants.vcf.gz" | wc -l)"
log "完成：$OUT/common_variants.vcf.gz"
```

### 02、cellanp
```
#!/bin/bash
# 用途：对单个文库的完整 BAM，在候选 SNP 位点上做 pileup，
# 统计每个细胞在各位点的 ref/alt 等位基因计数（AD/DP 矩阵），供 vireo 拆分
set -euo pipefail

# ---------- 参数 ----------
SAMPLE=$1
export PATH=/home/YangWenyan/miniconda3/envs/cellsnp/bin:$PATH
COUNT=/tmpdata/YangWenyan/yang-v2/count_all_v436
OUT=$COUNT/splitsample
BAM=$COUNT/$SAMPLE/outs/possorted_genome_bam.bam     # cellranger 位置排序 BAM
BARCODES=$OUT/$SAMPLE/barcodes.tsv                   # 过滤后的细胞 barcode 列表
REGIONS=$OUT/common_variants/common_variants.vcf.gz  # 候选 SNP 位点（02 脚本产物）

log() { echo "[$(date '+%F %T')] $*"; }

# ---------- pileup ----------
log "cellsnp-lite 开始：$SAMPLE"

# --cellTAG CB：BAM 中细胞 barcode 标签;--UMItag UB ：BAM 中 UMI 标签，去重后计数;--minMAF 0.1：保留小等位基因频率 >= 0.1 的位点;--minCOUNT 20：细胞内最低等位基因计数
cellsnp-lite -s "$BAM" -b "$BARCODES" -O "$OUT/$SAMPLE/cellsnp" \
  -R "$REGIONS" --cellTAG CB --UMItag UB --minMAF 0.1 --minCOUNT 20 -p 20 \
  | tee "$OUT/logs/03_cellsnp_${SAMPLE}.log"

log "cellsnp-lite 完成：$SAMPLE"
```

### 03、vireo
```
#!/bin/bash
# 用途：基于 cellsnp-lite 的等位基因矩阵做贝叶斯个体拆分。
# 本批次两个文库各自混了 3 条不同的幼体（共 6 条），故 -N 3 独立拆分
set -euo pipefail

# ---------- 参数 ----------
SAMPLE=$1
export PATH=/home/YangWenyan/miniconda3/envs/cellsnp/bin:$PATH
OUT=/tmpdata/YangWenyan/yang-v2/count_all_v436/splitsample

log() { echo "[$(date '+%F %T')] $*"; }

# ---------- 拆分 ----------
log "vireo 开始：$SAMPLE"
# -N 3：池中个体数;--randSeed 42：随机种子，保证可重复
vireo -c "$OUT/$SAMPLE/cellsnp" -N 3 -o "$OUT/$SAMPLE/vireo" \
  -p 20 --randSeed 42 | tee "$OUT/logs/04_vireo_${SAMPLE}.log"
log "vireo 完成：$SAMPLE"
```
