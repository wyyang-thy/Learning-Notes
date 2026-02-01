### record some analysis steps of Seurat here for myself to reveiew.
### 必须先了解一下seurat对象是什么样的格式
```
Seurat 对象（pbmc）
├── 原始数据（counts）          ← 抽屉1：测序得到的原始计数
├── 标准化数据（data）          ← 抽屉2：处理后的数据
├── 缩放数据（scale.data）      ← 抽屉3：进一步处理的数据
├── 细胞信息（meta.data）       ← 抽屉4：每个细胞的属性
├── 降维结果（reductions）      ← 抽屉5：PCA、UMAP等
├── 基因信息（features）        ← 抽屉6：基因的属性
└── 聚类结果（idents）          ← 抽屉7：细胞分群信息
```
```
pbmc
├── RNA assay
│   ├── counts [18000 x 5000]
│   ├── data [18000 x 5000]
│   ├── scale.data [2000 x 5000]  ← 新增（只有高变基因）
│   └── var.features: 2000 genes   ← 新增
├── meta.data [5000 x 6]
│   ├── orig.ident
│   ├── nCount_RNA
│   ├── nFeature_RNA
│   ├── percent.mt
│   ├── RNA_snn_res.0.8
│   └── seurat_clusters  ← 新增
└── reductions
    ├── pca [5000 x 50]   ← 新增
    └── umap [5000 x 2]   ← 新增
```

### 1、Assays（数据层） - 最核心的部分
```
# 查看有哪些 assay
names(pbmc@assays)
# 输出：[1] "RNA"

# RNA assay 包含三层数据
pbmc@assays$RNA@counts       # 原始计数（不可修改），差异表达分析、导出原始数据，不可修改这个挺好的，不管做成什么样都能重新开始。
pbmc@assays$RNA@data         # 标准化后的数据，可视化（Feature plot等）、计算表达量，已标准化，可比较。
pbmc@assays$RNA@scale.data   # 缩放后的数据（用于PCA等），PCA、聚类、热图，去除了批次效应，变量在同一尺度。
```

### 2、meta.data（细胞信息） - QC 的关键
```
# 查看 meta.data
head(pbmc@meta.data)

输出示例：
                     orig.ident nCount_RNA nFeature_RNA percent.mt
AAACCCAAGAAACCCA-1  lamprey_pbmc       5234         1523       3.2
AAACCCAAGAAACTGG-1  lamprey_pbmc       3892         1205       5.1
AAACCCAAGAACAATC-1  lamprey_pbmc       4521         1687       2.8

# orig.ident：样本来源（项目名称）
# nCount_RNA：每个细胞的总 UMI 数
# nFeature_RNA：每个细胞检测到的基因数
## 以下是在分析过程中添加进去的：
# percent.mt：线粒体基因比例（手动计算添加）
# seurat_clusters：聚类结果
# RNA_snn_res.0.8：不同分辨率的聚类结果
```
### 3、reductions（降维结果）
```
pbmc@reductions$pca@cell.embeddings   # 每个细胞的坐标
pbmc@reductions$pca@feature.loadings  # 每个基因的权重
pbmc@reductions$pca@stdev             # 标准差（用于 Elbow plot）

# 简洁写法
Embeddings(pbmc, reduction = "pca")
Loadings(pbmc, reduction = "pca")
```

### 在使用的时候为了避免$和@分不清用哪个，尽量使用函数调用
```
GetAssayData(pbmc, slot = "counts")
Embeddings(pbmc, reduction = "pca")
Idents(pbmc)
```

### GetAssayData() - 获取表达数据，从 Seurat 对象中提取基因表达矩阵，slot - 指定要提取哪一层数据，返回一个矩阵（行=基因，列=细胞）：
```
# 1. 提取原始计数（raw counts）
counts <- GetAssayData(pbmc, slot = "counts")

# 2. 提取标准化数据（normalized data）
data <- GetAssayData(pbmc, slot = "data")

# 3. 提取缩放数据（scaled data）
scale <- GetAssayData(pbmc, slot = "scale.data")
```

### Embeddings() - 获取降维坐标，提取降维后的细胞坐标（PCA、UMAP、tSNE 等），返回一个矩阵（行=细胞，列=维度）：
```
# 1. 提取 PCA 坐标
pca_coords <- Embeddings(pbmc, reduction = "pca")

# 2. 提取 UMAP 坐标
umap_coords <- Embeddings(pbmc, reduction = "umap")

# 3. 提取 tSNE 坐标
tsne_coords <- Embeddings(pbmc, reduction = "tsne")
```

### Idents() - 获取/设置细胞身份，获取或设置每个细胞的身份标签（聚类、细胞类型等），默认情况下，Idents 就是 seurat_clusters！！！
```
# 获取
Idents(object)

# 设置
Idents(object) <- new_idents
```
```
# 重命名
# 当前聚类是数字 0, 1, 2, 3, ...
levels(Idents(pbmc))
# [1] "0" "1" "2" "3" "4" "5" "6" "7"

# 根据 marker 基因，识别出了细胞类型
new.cluster.ids <- c(
  "T_cell",        # 聚类 0
  "B_cell",        # 聚类 1
  "Monocyte",      # 聚类 2
  "NK_cell",       # 聚类 3
  "DC",            # 聚类 4
  "Macrophage",    # 聚类 5
  "Neutrophil",    # 聚类 6
  "Platelet"       # 聚类 7
)

# 重命名
names(new.cluster.ids) <- levels(pbmc)
pbmc <- RenameIdents(pbmc, new.cluster.ids)

# 现在查看
head(Idents(pbmc))
```

输出：
```
AAACCCAAGAAACCCA-1 AAACCCAAGAAACTGG-1 AAACCCAAGAACAATC-1 
            T_cell             Monocyte               B_cell 
Levels: T_cell B_cell Monocyte NK_cell DC Macrophage Neutrophil Platelet
```
