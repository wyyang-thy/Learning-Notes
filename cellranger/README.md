# cellranger学习记录
## 这里记录我第一次跑Cellranger全流程，也会记录一些我新学习到的知识，我想应该和前边的杂合知识分开记录，另外，因为里边记载了太多字来帮助我理解，所以真正需要运行的代码以###来开头标注，而且为了方便我下次复习，全部使用绝对路径形式。

### 首先在集群上申请计算资源，不能在登陆节点上运行作业
```
srun -p cpu -n 64 --pty /bin/bash
```

### 创立一个链接，-s是symbolic表示软链接，对于目录的链接只能使用软链接，可以跨目录，删除原文件链接失效。硬链接是给同一个东西取了不同的名字无法跨越文件分区，删除原文件链接不会失效
```
ln -s /lustre/home/acct-medcl/share /lustre/home/acct-medcl/wyyang2025/workspace/share_by_teacherCL
```
## 语法：ln -s 原文件目录 目的文件目录

## 记录一下10X illumina的下机数据是什么样的，
 - 首先是两个个fastq文件，
 - read1文件是barcode+UMI；
 - read2文件是5'端开始的序列信息。
 - 每个文件的第一行是文件名字，第二行是碱基序列，第三行记录是正义链（+）还是反义链（-），第四行记录的质量信息

# 第一步：数据质量控制QC
## 1.1 检查 FASTQ 文件完整性
### 检查文件是否完整（通过 md5 校验）
```
md5sum -c md5sum.txt
```
### 查看文件大小→27G
```
ls -lh TF2212CLCL1X_S1_L003_R1_001.fastq.gz
```
### 简单查看前几条序列，这里需要记住查看fastq.gz这种压缩文件不可以直接使用head和vim，只能用zcat来查看！！！或者在其他查看命令前加上字母z
```
zcat TF2212CLCL1X_S1_L003_R1_001.fastq.gz | head -n 20
```
## 1.2 使用 FastQC 进行质量评估
### 这里需要注意的是在集群上conda需要load不能直接用
```
srun -p small -n 4 --pty /bin/bash
module load miniconda3/4.8.2
```
```
### 安装 FastQC
conda install -c bioconda fastqc
```
### 运行质量控制，这个我没运行，因为我没有成功加载miniconda，这里的意思大概就是说可以同时质量控制不止一个文件
```
fastqc sample_R1.fastq.gz sample_R2.fastq.gz -o qc_results/
```
### 查看生成的 HTML 报告
```
#qc_results/sample_R1_fastqc.html
```
## 1.3 使用 MultiQC 汇总多个样本
### 安装 MultiQC
```
conda install -c bioconda multiqc
``` 
### 汇总所有 FastQC 报告
```
multiqc qc_results/ -o multiqc_report/
```
# 第二步：预处理，这一步才是比较重要的！！！！！复习时可以直接从这一步看起！！！！！
## 2.1 10x Genomics 数据：使用 Cell Ranger
## Cell Ranger 是 10x 官方工具，功能包括：比对到参考基因组；细胞条形码识别（区分真实细胞和空液滴）；UMI 去重（分子计数）；生成表达矩阵

### 加载cellranger工具，需要使用什么工具但是发现不能直接使用的时候直接去hpc studio手册里查，一般都有。
```
module load cellranger/7.2.0
```
### /lustre/home/acct-medcl/wyyang2025/workspace/share_by_teacherCL/share/lamprey/pmar_ref这个路径下存放了GCF_010993605.1_kPetMar1.pri_genomic.fna和 GCF_010993605.1_kPetMar1.pri_genomic.gtf这两个文件
### 首先需要知道fasta文件是什么，这个基因组参考序列是经由reads拼装组成的染色体和线粒体序列，其中以>开头的是染色体/线粒体的名字，其后是对应的序列。相当于一本书的正文内容。
### 对应的gtf文件是存储基因注释信息的文本格式，其中包含每个基因在染色体上的位置；每个基因包含哪些外显子（exon）；基因的起始和终止位置；基因名称、ID 等信息。相当于一本书的目录。
### head -n 50 GCF_010993605.1_kPetMar1.pri_genomic.fna以及head -n 50 GCF_010993605.1_kPetMar1.pri_genomic.gtf可以进行查看其中的内容
### 这两个文件是原始的、人类可读的文本文件，cellranger不能直接使用这些原始文件，它需要序列快速对比索引，预处理的基因注释，特定的目录结构和文件格式，所以需要构建参考基因组，使用cellranger mkref这个命令。
### 这个命令使用 STAR 软件建立基因组索引，这样比对时可以快速找到序列匹配位置，然后解析 GTF 文件，提取外显子（exon）信息，建立基因ID到基因名的映射，过滤无效的注释，最后创建 Cell Ranger 专用的目录结构，类似于：
```
pmar_ref/
├── fasta/
│   └── genome.fa          # 处理后的基因组序列
├── genes/
│   └── genes.gtf          # 处理后的注释
├── star/                   # STAR 索引文件
│   ├── SA
│   ├── SAindex
│   └── ...
└── reference.json         # 元数据
```
## 实际上这些操作是可以实现的，但是必须在自己有权限的路径中操作，这时候必须从share文件夹中出来
### 我建立了一个新的文件夹来存储参考基因组
```
mkdir /lustre/home/acct-medcl/wyyang2025/workspace/share_by_teacherCL/lamprey_reference
cd /lustre/home/acct-medcl/wyyang2025/workspace/share_by_teacherCL/lamprey_reference  #以下操作都在这个路径下进行嗷
```
### 问题是gtf文件中的gene_id列会存在空格也就是gene_id ""这样的形式导致mkref没办法直接运行，所以需要删掉这些空白信息，而且在cellranger官网上也建议在mkref前先运行 mkgtf 预过滤
### 需要找的是 gene_id ""。因为命令本身已经用了双引号包裹，为了让 Linux 明白中间那两个双引号也是我们要找的内容，所以加了转义符 \。这里路径都是一个，太长了就不写绝对路径了...
### grep中的-v参数：是 “反向选择（Invert-match）”。通常 grep 是找“包含某词的行”，加了 -v 之后，它会删掉包含某词的行，留下剩下的。所以这里是删掉包含空值的行，留下正常的。
```
cat /lustre/home/acct-medcl/wyyang2025/workspace/share_by_teacherCL/share/lamprey/pmar_ref/GCF_010993605.1_kPetMar1.pri_genomic.gtf | grep -v "gene_id \"\"" > GCF_010993605.1_kPetMar1.pri_genomic.nonEmpty.gtf  
```
### 指定只保留 protein_coding（蛋白编码基因）和 lncRNA（长链非编码 RNA）。并且把GCF_010993605.1_kPetMar1.pri_genomic.nonEmpty.gtf这个文件中的这些信息写入GCF_010993605.1_kPetMar1.pri_genomic.fil.gtf中
```
cellranger mkgtf GCF_010993605.1_kPetMar1.pri_genomic.nonEmpty.gtf GCF_010993605.1_kPetMar1.pri_genomic.fil.gtf \   
  --attribute=gene_biotype:protein_coding \
  --attribute=gene_biotype:lncRNA
```
### 删掉临时存储的文件GCF_010993605.1_kPetMar1.pri_genomic.nonEmpty.gtf
```
rm -rf GCF_010993605.1_kPetMar1.pri_genomic.nonEmpty.gtf
```
### 构建参考基因组,会在当前目录下生成pmar_ref/文件夹，这就是Cell Ranger需要的参考基因组。
```
cellranger mkref \
  --genome=pmar_ref \  #输出的参考基因组名称，注意这个没法用绝对路径
  --fasta=/lustre/home/acct-medcl/wyyang2025/workspace/share_by_teacherCL/share/lamprey/pmar_ref/GCF_010993605.1_kPetMar1.pri_genomic.fna \  #基因组序列文件（.fna）
  --genes=/lustre/home/acct-medcl/wyyang2025/workspace/cellranger_reference/GCF_010993605.1_kPetMar1.pri_genomic.fil.gtf \  #基因注释文件（.gtf），是上一步处理过的
  --nthreads=16 \  #使用的线程数
  --memgb=64  #使用的内存（GB）
```
### 接下来就是运行cellranger！！！！！
```
cellranger count \
  --id=pbmc \  #Cell Ranger 会在当前目录下创建一个名为 pbmc 的文件夹，所有的中间文件和最终结果（如 web_summary.html、表达矩阵等）都会存放在这个文件夹里。
  --transcriptome=/lustre/home/acct-medcl/wyyang2025/workspace/cellranger_reference/pmar_ref \  #指定参考基因组索引的路径。
  --fastqs=/lustre/home/acct-medcl/wyyang2025/workspace/share_by_teacherCL/share/lamprey/lamprey_data_202212SC_pbmcNsb/raw1 \  #测序原始文件（FASTQ）所在的文件夹路径。
  --include-introns=true \  #将比对到内含子（Introns）区域的 Reads 也计入基因表达量。
  --nosecondary \  #跳过二级分析。Cell Ranger 默认会自动进行聚类（Clustering）和差异表达分析。但实际上我们后续会自己分析。
  --disable-ui \  #禁用网页端的实时监控界面。由于在集群上运行，通常无法直接通过浏览器访问计算节点的 IP（比如超算的这种http://cas101...链接）。禁用它可以减少一些不必要的网络开销。
  --localcores=16 \  #限制程序最多使用的 CPU 核心数为 16 个。
  --localmem=100  #限制程序最多使用的内存为 100GB。
```
