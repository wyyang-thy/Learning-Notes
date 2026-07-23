# 20260723深入学习单细胞分析
## 1、完成标准 Seurat PBMC pipeline：读入、QC、过滤、Normalize、PCA、UMAP、cluster、marker、注释。
```
# 下载了seurat的官方pbmc数据传到超算的对应位置
 rsync -avhP /mnt/d/DOWNLOADS/pbmc3k_filtered_gene_bc_matrices.tar.gz wyyang2025@data.hpc.sjtu.edu.cn:/lustre/home/acct-medcl/wyyang2025/workspace/seurat_learn/raw
# 解压这个压缩文件，其中x是extract是提取解压的意思，z是处理gzip格式，v是verbose显示进程，f是filename指定文件名称的意思，对应于压缩就是-czvf，其中c是create创建一个压缩包的意思
tar -xzvf pbmc3k_filtered_gene_bc_matrices.tar.gz
# /lustre/home/acct-medcl/wyyang2025/workspace/seurat_learn/raw/filtered_gene_bc_matrices/hg19得到这个路径下就是标准的处理格式，有barcode、gene、matrix三个文件
```
