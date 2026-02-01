### 记录我学习过程中学习到的神奇的R函数和一些用法

### list.files函数可以对路径中的文件名字进行按需要查找，一般只需要指定前两个参数，第二个是正则表达式，在R中.表示任何字符所以这里需要用转义字符\，但在R中还需要用两个\来进行转义，第三个参数表示是否需要带路径的全名（读取文件时通常需要 TRUE）。
```
list.files(path = "E:/meta/DEseq2", pattern = "\\.rds$", full.names = TRUE)
files <- list.files(pattern = "_normal_vs_disordered\\.rds$")# 返回到变量files中，存放的都是符合要求的文件名字
```

### str_detect函数检测字符串中是否包含某个模式，返回 TRUE 或 FALSE。常配合逻辑非 ! 使用。
```
files <- files[!str_detect(files, "obj|enrichGO")]# 只保留没有出现obj和enrichGO的文件名字并更新变量files
```

### str_remove函数把字符串里匹配到的部分删掉（替换为空）。
```
cell_type_name <- str_remove(f, "_normal_vs_disordered\\.rds")# # f 是文件名，比如 "B_naive_normal_vs_disordered.rds"。用 str_remove 把后面那一长串后缀删掉。结果：cell_type_name 变成了 "B_naive"。
```

### map_dfr函数，purrr包中的函数，可以看作是两个函数来用，首先是map函数，它可以将一群文件逐个传入一个函数中进行处理，处理之后由dfr函数合并成大的数据框，dfr指的是dataframe row bind
# map函数会将files逐个传入函数function中处理，每个传入的文件都是f，这里是一个隐形函数，因为并没有定义一个实际的名字，用完即结束，最后返回一个列表
```
all_deg_df <- map_dfr(files, function(f) {
  
  # A. 提取细胞类型名
  # f 是文件名，比如 "B_naive_normal_vs_disordered.rds"。用 str_remove 把后面那一长串后缀删掉。结果：cell_type_name 变成了 "B_naive"。
  cell_type_name <- str_remove(f, "_normal_vs_disordered\\.rds")
  
  # B. 读取数据，因为传入的f实际上是一个文件名，而在最开始设定了路径，所以这里只需要传入一个文件名就可以读取
  temp_data <- readRDS(f) 
  df <- as.data.frame(temp_data)
  
  # C. 检查是否已经有 "gene" 这一列
  # 如果没有 "gene" 列，才把行名转为 gene 列；如果已经有了，就直接用
  if (!"gene" %in% names(df)) {
    df <- rownames_to_column(df, "gene")
  }
  
  # D. 整理数据：只拿走我们需要的四列，这个代码申请的地方！！！只处理到这里不用任何合并操作，最后就会把所有文件的这四列合并在一起
  df %>% 
    mutate(Cell_Type = cell_type_name) %>% 
    select(gene, Cell_Type, log2FoldChange, padj) 
})
```
