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
