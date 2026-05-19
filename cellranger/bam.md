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
