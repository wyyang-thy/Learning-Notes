---
title: "Seurat PBMC Analysis Pipeline Learning"
author: "WYYang"
date: "2026-02-02"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
# 设置工作目录为当前项目文件夹 (不需要写绝对路径，方便在不同电脑上运行)
# knitr会自动将.Rmd文件所在的目录作为工作目录
```

## 1. 环境加载与数据读取

加载单细胞分析所需的必要包。

```{r library, message=FALSE, warning=FALSE}
library(dplyr)
library(Seurat)
library(patchwork)
```

读取 10X Genomics 输出的数据。

```{r load_data}
# 使用相对路径，避免硬编码服务器绝对路径
data_dir <- "./data/pbmc/outs/filtered_feature_bc_matrix/" 

# 如果是练习用的 pbmc3k 数据，Seurat 也可以直接下载，但在服务器上通常读取本地文件
pbmc.data <- Read10X(data.dir = data_dir)

# 创建 Seurat 对象，初步质检
# min.cells = 3: 一个基因至少在3个细胞中表达才保留
# min.features = 200: 一个细胞至少检测到200个基因才保留
pbmc <- CreateSeuratObject(counts = pbmc.data, project = "pbmc3k", min.cells = 3, min.features = 200)

pbmc
head(rownames(pbmc))
```

## 2. 质量控制 (QC)

我们需要剔除低质量的细胞（如死细胞或空液滴）。

* **nFeature_RNA**: 细胞内检测到的基因数量（过低可能是空液滴，过高可能是双细胞/Doublet）。
* **percent.mt**: 线粒体基因比例（过高通常意味着细胞正在死亡，胞质RNA泄漏，只剩下线粒体RNA）。

```{r qc}
# 计算线粒体比例 (Human基因通常以 MT- 开头，Mouse 以 mt- 开头，Lamprey 可能不同，所以在这里我并没有检测出任何线粒体数据)
pbmc[["percent.mt"]] <- PercentageFeatureSet(pbmc, pattern = "^MT-")

# 可视化 QC 指标（可选）
VlnPlot(pbmc, features = c("nFeature_RNA", "nCount_RNA", "percent.mt"), ncol = 3)

# 过滤：保留基因数在 200-2500 之间，且线粒体比例小于 5% 的细胞
pbmc <- subset(pbmc, subset = nFeature_RNA > 200 & nFeature_RNA < 2500 & percent.mt < 5)
```

## 3. 标准化与高变基因查找

```{r normalize_find_variable}
# 标准化：消除测序深度不同带来的影响 (LogNormalize)
pbmc <- NormalizeData(pbmc, normalization.method = "LogNormalize", scale.factor = 10000)

# 寻找高变基因 (Variable Features)
# 目的：找出在不同细胞间表达差异最大的基因（包含生物学信号的基因），用于后续降维。
pbmc <- FindVariableFeatures(pbmc, selection.method = "vst", nfeatures = 2000)

# 查看前10个高变基因
top10 <- head(VariableFeatures(pbmc), 10)
top10

# 可视化高变基因
plot1 <- VariableFeaturePlot(pbmc)
plot2 <- LabelPoints(plot = plot1, points = top10, repel = TRUE)
plot1 + plot2
```

## 4. 数据归一化 (Scaling)

这一步将每个基因在所有细胞中的表达量进行 **中心化 (Mean=0)** 和 **标准化 (Variance=1)**。
* **目的**：使所有基因具有相同的“权重”，防止高表达基因（如管家基因）掩盖了低表达基因的信号。

```{r scale}
all.genes <- rownames(pbmc)
pbmc <- ScaleData(pbmc, features = all.genes)
```

## 5. 线性降维 (PCA)

使用高变基因进行主成分分析。我们不需要使用所有基因，因为大部分基因（如基础代谢基因）在所有细胞里表达都很稳定，对区分细胞类型没有贡献。

```{r pca}
pbmc <- RunPCA(pbmc, features = VariableFeatures(object = pbmc))

# 肘部图 (Elbow Plot)：帮助决定使用多少个 PC 用于后续分析
ElbowPlot(pbmc)
```

## 6. 非线性降维可视化 (UMAP/t-SNE)

PCA 将数据压缩到 20 维，UMAP 进一步将其压缩到 2 维平面，以便人眼观察。
UMAP 的核心逻辑：保证在 20 维空间里靠得近的细胞，在 2 维平面上也画得近；把不相似的细胞推开。
* **这里其实是有一个小巧思在的，因为seurat官网上的步骤实际上是先PCA然后马上细胞聚类，随后再进行非线性降维，这样会导致某些簇的点会混在一起，导致不consistent，但是如果先降维再聚类就会比较干净！！！**

```{r umap}
pbmc <- RunUMAP(pbmc, reduction = "pca", dims = 1:20)
```

## 7. 细胞聚类 (Clustering)

基于 PCA 降维后的空间（这里取前20个 PC）构建细胞间的近邻关系网（SNN Graph），然后使用 Louvain 算法切分群落。

```{r cluster}
# FindNeighbors: 在高维空间里给每个细胞找邻居
# 如果想要按照上一步的小tips做的话，需要把reduction改为umap，dims改为1:2，但是这么做的话只是利用了二维的数据信息，相比于使用pca来说会丢失18维的信息，也比较容易理解，在二维上聚类就比较容易把细胞群分的比较开，在高维上聚类肯定就容易混杂。
* **聚类是基于 PCA 降维（前 20 个主成分）进行的，以保留高维生物学距离。UMAP 投影上的轻微群落混合反映了连续的生物学转变，而非技术假象。**
pbmc <- FindNeighbors(pbmc, reduction = "pca", dims = 1:20)

# FindClusters: 切分“朋友圈”
# resolution 参数控制分群的粗细：值越大，群越多越细
# 小tips：先把群分多一点，把有差异的细胞群先展示出来，随后再根据marker进行合并相似的细胞群。
pbmc <- FindClusters(pbmc, resolution = 0.6)

# 查看前 5 个细胞的 Cluster ID
head(Idents(pbmc), 5)

# 可视化聚类结果
DimPlot(pbmc, reduction = "umap", label = TRUE)
```

## 8. 保存结果

```{r save}
# 检查 output 文件夹是否存在，不存在则创建
if(!dir.exists("output")){
  dir.create("output")
}

saveRDS(pbmc, file = "output/pbmc_tutorial.rds")
```
首先cellranger的标准是一万细胞左右，但是这个实际得到的细胞是两万多，所以这里会有很多doublet
另外，如果说一个细胞个头很小则说明它的UMI很少
所以我想有必要记录一下cellranger结果的代表意义，其中一行代表一个基因，一列代表一个细胞，对应的数值是去重后的UMI值也就是原始的mRNA数量，表示这个基因在这个细胞中有多少转录本，所以UMI值在一定意义上是可以代表细胞大小的。
