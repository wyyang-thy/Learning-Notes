# 首先需要明确的是我们做的单细胞测序实际上是测量的RNA，油包水的时候吸附的mRNA会反转录成cDNA，所以我们后续得到的也都是DNA序列，bam文件里存在的是每条read的碱基序列和测序质量值、比对到了基因组的哪个染色体上的哪个位置、比对的准不准、属于哪个细胞/RNA
# 老师今天说二代测序在实验上很难出错
# 20260519针对cellranger结果比对到参考基因组以及转录组上的比率太低尝试找原因
# 我现在做的这个选择1是从bam中找到没有匹配到基因组上的reads然后转换成fasta再取前100条序列去做blast
## 先申请计算节点
```
srun -p cpu -n 64 --pty /bin/bash
```

## 加载samtools
```
module load bwa samtools
```

## 用32个线程提取所有未比对(-f4)到七鳃鳗基因组的reads
```
samtools view -@ 32 -b -f 4 possorted_genome_bam.bam > unmapped.bam
```

## 将 BAM 转换为 FASTA 格式
```
samtools fasta -@ 32 unmapped.bam > unmapped.fasta
```

## 抽取前 100 条序列（FASTA 格式每条序列占 2 行，所以取 200 行）
```
head -n 200 unmapped.fasta > unmapped_sample_100.fasta
```
## 把这个fasta文件下载到本地去ncbi上跑blast
## 没办法直接在计算节点进行运行，我就写了一个脚本
```
#!/bin/bash
#SBATCH --job-name=get_unmapped      # 作业名称
#SBATCH --partition=cpu              # 提交的队列名称
#SBATCH --nodes=1                    # 申请 1 个计算节点
#SBATCH --ntasks=1                   # 运行 1 个任务
#SBATCH --cpus-per-task=16           # 为这个任务统一申请 16 个 CPU 核心
#SBATCH --output=samtools_%j.out     # 正常输出日志
#SBATCH --error=samtools_%j.err      # 错误报错日志

# 加载工具
module load bwa samtools

# 记录任务开始时间
echo "Job started at: $(date)"

# 步骤 1: 提取未比对 Reads。统一使用 15 个线程（留 1 个给系统 I/O 调度，最稳妥）
echo "步骤 1: 正在从 BAM 中提取未比对到参考基因组的 Reads..."
samtools view -@ 15 -b -f 4 possorted_genome_bam.bam > unmapped.bam

# 步骤 2: BAM 转 FASTA。同样拉高线程数，充分利用申请的 16 个核心
echo "步骤 2: 正在将 BAM 文件转换为 FASTA 格式..."
samtools fasta -@ 15 unmapped.bam > unmapped.fasta

# 步骤 3: 抽样 100 条序列
echo "步骤 3: 正在抽取前 100 条序列用于 NCBI BLAST 抽样..."
head -n 200 unmapped.fasta > unmapped_sample_100.fasta

# 记录任务结束时间
echo "Job finished at: $(date)"
```
## 接下来用这个序列去ncbi上做blastn，结果没有发现别的物种的信息
## 师兄建议我用未必对上的序列和t2t参考基因组做blast
## 先下载blast软件
```
wget http://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/2.13.0/ncbi-blast-2.13.0+-x64-linux.tar.gz
```
## 解压
```
tar -zxvf ncbi-blast-2.13.0+-x64-linux.tar.gz
```
## 添加环境变量
```
export PATH=/lustre/home/acct-medcl/wyyang2025/software/blast/ncbi-blast-2.13.0+/bin:$PATH
```
## 写脚本
```
#!/bin/bash
#SBATCH --job-name=lamprey_blast     # 你的作业名称
#SBATCH --partition=cpu              # 提交到 cpu 队列
#SBATCH --nodes=1                    # 申请 1 个计算节点
#SBATCH --ntasks=1                   # 运行 1 个主要任务
#SBATCH --cpus-per-task=32           # 满载 64 核算力
#SBATCH --output=blast_run_%j.out    # 实时标准输出日志
#SBATCH --error=blast_run_%j.err     # 报错与进度日志

export PATH=/lustre/home/acct-medcl/wyyang2025/software/blast/ncbi-blast-2.13.0+/bin:$PATH

echo "Checking blastn location in compute node:"
which blastn
blastn -version | head -n 1

# 七鳃鳗基因组精确绝对路径
GENOME_FA="/lustre/home/acct-medcl/wyyang2025/workspace/cellranger_reference/t2t/lamprey_t2t_v1/fasta/genome.fa"

# 未比对大 FASTA 文件
QUERY_FASTA="/lustre/home/acct-medcl/wyyang2025/workspace/Accuramed20260228/cellranger_by_myself/CR10_20260113-V4-5/outs/unmapped.fasta"

# 当前工作输出目录
WORK_DIR=$(pwd)

# -----------------------------------------------------------------
# 构建七鳃鳗基因组本地 BLAST 数据库
# -----------------------------------------------------------------
echo "Step 1: Building local BLAST database..."
# 建库索引文件会直接生成在你当前的工作目录下，名字叫 lamprey_t2t_db
makeblastdb \
  -in "$GENOME_FA" \
  -dbtype nucl \
  -out "${WORK_DIR}/lamprey_t2t_db"

echo "Database build completed."
echo "-----------------------------------------"

# -----------------------------------------------------------------
# 执行高并发批量 BLASTN 比对
# -----------------------------------------------------------------
echo "Step 2: Starting massive BLASTN alignment..."
echo "This will take some time because unmapped.fasta is HUGE."

blastn \
  -query "$QUERY_FASTA" \
  -db "${WORK_DIR}/lamprey_t2t_db" \
  -out "${WORK_DIR}/blast_results.tsv" \
  -evalue 1e-5 \
  -outfmt "6 qseqid sseqid pident length mismatch gapopen qstart qend sstart send evalue bitscore" \
  -num_threads 30

echo "========================================="
echo "Job finished successfully at: $(date)"
echo "========================================="
```
## 利好消息：这些unmapped的序列能够通过blast比对上t2t参考基因组，只是因为一条序列会比对到参考基因组的多个不同地方，不是唯一的比对，在cellranger里边为了保持唯一性会把这些多重比对的序列算作unmapped
## 接下来我可以通过调节cellranger参数或者使用star来进行新的mapping
### 加入了一个新的参数也就是--multi-mapper=EM，允许多重匹配，并且gemini建议我采用比对内含子，实际使用上不行，10.0.0的cellranger并没有这个参数
### 我现在的实际情况是有相当一部分的序列并没有比对到参考基因组上，我把没有比对到参考基因组上的序列去ncbi上做blast，超过60%的序列都是核糖体基因序列，于是我用这些未必对的序列跟t2t参考基因组做blast，基本上每一条未必对序列都能够匹配到参考基因组上的多个位置包括手脚架和正式的染色体序列，这也有可能是被cellranger列为unmapped的原因，而且比如比对到chr80的序列实际上在gtf上的注释是非常完善的，于是gemini建议我使用cellranger multi
```
vim multi_config.csv
[gene-expression]
reference,/lustre/home/acct-medcl/wyyang2025/workspace/cellranger_reference/t2t/lamprey_t2t_v1
include-introns,true
create-bam,true

[libraries]
fastq_id,fastqs,feature_types
20260113-V4-5,/lustre/home/acct-medcl/wyyang2025/workspace/Accuramed20260228/raw_data/20260113-V4-5,Gene Expression
```
### 脚本
```
vim run_multi.sh
#!/bin/bash
#SBATCH --job-name=20260113-V4-5-T2T_MultiMode
#SBATCH --partition=cpu
#SBATCH --output=res_multi_%j.log
#SBATCH --error=err_multi_%j.log
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=16
#SBATCH --mem=100G
#SBATCH --time=7-00:00:00

set -euo pipefail

module purge

CELLRANGER_HOME=/lustre/home/acct-medcl/wyyang2025/software/cellranger-10.0.0
export PATH="$CELLRANGER_HOME/bin:$PATH"
export TENX_IGNORE_DEPRECATED_OS=1

OUT_BASE=/lustre/home/acct-medcl/wyyang2025/workspace/Accuramed20260228/cellranger_by_myself

mkdir -p "$OUT_BASE"
cd "$OUT_BASE" || exit 1

echo "Cell Ranger Multi Version: $(cellranger --version)"

# 🚀 启动官方高级多组学/多重比对分流拯救模式
cellranger multi \
  --id="CR10_T2T_MultiMode_V4-5" \
  --csv="./multi_config.csv" \
  --localcores=16 \
  --localmem=90
```
## 结果是相对于cellranger count有轻微的提高，原来能准确匹配到转录组上的比对率大概30%，使用multi之后提高到了36%，比对到参考基因组上有61%，我觉得还是比较低所以使用了star进行比对
### 先建立索引
```
#!/bin/bash
#SBATCH --job-name=lamprey_star_index_v11
#SBATCH --partition=cpu
#SBATCH -N 1
#SBATCH --ntasks-per-node=40
#SBATCH --exclusive
#SBATCH --output=index_new_%j.out
#SBATCH --error=index_new_%j.err

set -euo pipefail

# 🎯 注入你刚刚纯手工验证成功的绝对路径
MY_STAR=/lustre/home/acct-medcl/wyyang2025/software/starr/STAR-2.7.11b/bin/Linux_x86_64_static/STAR

FASTA=/lustre/home/acct-medcl/wyyang2025/workspace/cellranger_reference/t2t/lamprey_t2t_v1/fasta/genome.fa
GTF=/lustre/home/acct-medcl/wyyang2025/workspace/cellranger_reference/t2t/lamprey.final.pasa.gtf

# 建立一个全新的高版本索引目录
OUT_DIR=/lustre/home/acct-medcl/wyyang2025/workspace/cellranger_reference/t2t/star_index_v11
mkdir -p "$OUT_DIR"

echo "Using Hand-made STAR Version: $($MY_STAR --version)"

# 🚀 40核独占火力全开，十几分钟即可冲完
$MY_STAR --runMode genomeGenerate \
  --genomeDir "$OUT_DIR" \
  --genomeFastaFiles "$FASTA" \
  --sjdbGTFfile "$GTF" \
  --sjdbOverhang 149 \
  --genomeSAindexNbases 12 \
  --runThreadN 40
```
### 进行比对
```
#!/bin/bash
#SBATCH --job-name=GeneMind_STARsolo
#SBATCH --partition=cpu
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=16
#SBATCH --mem=120G
#SBATCH --time=3-00:00:00

set -euo pipefail

# 🎯 锁定你纯手工最新版的独立物理路径
MY_STAR=/lustre/home/acct-medcl/wyyang2025/software/starr/STAR-2.7.11b/bin/Linux_x86_64_static/STAR

INDEX=/lustre/home/acct-medcl/wyyang2025/workspace/cellranger_reference/t2t/star_index_v11
FASTQ_DIR=/lustre/home/acct-medcl/wyyang2025/workspace/Accuramed20260228/raw_data/20260113-V4-5
OUT_BASE=/lustre/home/acct-medcl/wyyang2025/workspace/Accuramed20260228/starsolo_output

R2_FILES=$(ls $FASTQ_DIR/*_R2_*.fastq.gz | tr '\n' ',' | sed 's/,$//')
R1_FILES=$(ls $FASTQ_DIR/*_R1_*.fastq.gz | tr '\n' ',' | sed 's/,$//')

# 🎯 核心修正：换回你最初就写对的那个解压好的 737K 白名单！
WHITELIST=/lustre/home/acct-medcl/wyyang2025/software/cellranger-10.0.0/lib/python/cellranger/barcodes/737K-august-2016.txt

mkdir -p "$OUT_BASE"
cd "$OUT_BASE" || exit 1

echo "Running Authenticated STAR version: $($MY_STAR --version)"
echo "Starting STARsolo alignment for GeneMind PE150 Library..."

# 🚀 全力冲刺：精准物理坐标硬切真迈 Read 1，全面激活 EM 算法拯救 chr80
$MY_STAR --genomeDir "$INDEX" \
  --readFilesIn "$R2_FILES" "$R1_FILES" \
  --readFilesCommand zcat \
  --runThreadN 16 \
  --soloType CB_UMI_Simple \
  --soloCBwhitelist "$WHITELIST" \
  --soloCBstart 1 --soloCBlen 16 \
  --soloUMIstart 17 --soloUMIlen 12 \
  --soloBarcodeReadLength 0 \
  --soloCBmatchWLtype 1MM_multi \
  --soloFeatures Gene GeneFull \
  --soloMultiMappers EM \
  --outFilterMultimapNmax 100 \
  --outFileNamePrefix "$OUT_BASE/starsolo_"
```
### 唯一比对率53.37%，多重比对11.09%，所以大概有64%能比对到参考基因组上，未比对（序列太短）24.47%，但是问题是它无法正确识别barcode，而且相对于cellranger来说匹配到参考基因组上的比率并没有提高很多，仅仅提高了3%
```
# 先把这个巨大的sam文件转成bam文件然后我要删掉这个sam
srun -p cpu -n 64 --pty /bin/bash
module load bwa samtools
samtools view -S -b starsolo_Aligned.out.sam > starsolo_Aligned.out.bam
```
## 我去检查了gtf注释文件中是不是有核糖体基因注释，结果发现里面只有基因在染色体上的起止坐标，没有任何基因功能、名称或分类注释，没有核糖体基因
```
# 其中i表示支持大小写，E表示支持正则表达式匹配多个值
grep -i -E "ribosom|rRNA|RPL|RPS" lamprey.final.pasa.gtf | head -n 20
# awk '{print $3}'这个是把第三列打印到屏幕上
grep -v "^#" lamprey.final.pasa.gtf | awk '{print $3}' | sort | uniq -c
# 以下是结果
# 编码序列 467294 CDS
# 外显子 492591 exon
# 5'非翻译区  46680 five_prime_UTR
# 基因  25179 gene
# 3'非翻译区  37849 three_prime_UTR
# 转录本  41212 transcript
```
## 于是我决定采纳gemini的建议使用multi的结果，用scTE去识别bam文件中的转座子，采纳不了，我没有转座子的bed格式文件...
```
# 使用预先建立好的conda环境下载scTE工具
module load miniconda3
source activate /lustre/opt/condaenv/life_sci
conda list
cd /lustre/home/acct-medcl/wyyang2025/software
git clone https://github.com/JiekaiLab/scTE.git
cd scTE
python setup.py install --user
```
## 新方法
```
# 下载扫描重复序列的软件red
cd /lustre/home/acct-medcl/wyyang2025/software
wget https://github.com/BioinformaticsToolsmith/Red/archive/refs/heads/master.zip
unzip master.zip
cd /lustre/home/acct-medcl/wyyang2025/software/repetbed/Red-master/src_2.0
sed -i 's/g++-8/g++/g' Makefile
make clean
mkdir -p ../bin ../bin/exception ../bin/ms ../bin/nonltr ../bin/test ../bin/utility ../bin/tr
make
cd ..
mkdir -p genome_dir output_dir
# 把T2T的基因组建立一个软连接
ln -s /lustre/home/acct-medcl/wyyang2025/workspace/cellranger_reference/t2t/lamprey_t2t_v1/fasta/genome.fa genome_dir/
# 开始扫描重复序列
./bin/Red -gnm genome_dir -msk output_dir
./bin/Red -gnm genome_dir -rpt output_dir -frm 2
cat output_dir/*.bed | awk '{print $1"\t"$2"\t"$3"\tLamprey_Repeat\t.\t+"}' > /lustre/home/acct-medcl/wyyang2025/workspace/cellranger_reference/t2t/lamprey_repeats.bed
# 检查
head -n 5 /lustre/home/acct-medcl/wyyang2025/workspace/cellranger_reference/t2t/lamprey_repeats.bed
# 确保系统能找到scTE命令
export PATH="/lustre/home/acct-medcl/wyyang2025/.local/bin:$PATH"
cd /lustre/home/acct-medcl/wyyang2025/workspace/cellranger_reference/t2t/
sed -i "s/'rU'/'r'/g" /lustre/home/acct-medcl/wyyang2025/.local/lib/python3.12/site-packages/scTE-1.0-py3.12.egg/EGG-INFO/scripts/scTE*
sed -i 's/"rU"/"r"/g' /lustre/home/acct-medcl/wyyang2025/.local/lib/python3.12/site-packages/scTE-1.0-py3.12.egg/EGG-INFO/scripts/scTE*
/lustre/home/acct-medcl/wyyang2025/.local/bin/scTE_build -te lamprey_repeats.bed -gene lamprey.final.pasa.gtf -g other -o lamprey_t2t
```
### 这个失败了，不整了
