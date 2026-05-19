# 20260519针对cellranger结果比对到参考基因组以及转录组上的比率太低尝试找原因
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
