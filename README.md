# Learning-Notes
Record some of my learning processes.
# 我的学习记录

# 测序部分
### 一般的测序仪只能测长度有限制的、两端处理过的DNA
### 测序深度 = total read length/target genome length
### 测序广度 = 1 - 1 / e^测序深度
### 测序得到的fastQ文件可以做序列拼装、比对、注释
### 拼装assembly：实际上是测序的逆过程，由大量碎片化的Reads还原回target genome的过程，因为拼装的结果并不能完全和目标基因组一样，会有一些间断，把这样的拼装结果叫做contigs，双端测序时可以把contigs锚定成scaffolds脚手架，N50值用来评价拼装的好坏
### mapping/alignment序列比对：最重要的部分！把read比对到参考基因组上，但是由于测序错误、repeat片段故容易出错。cellranger
### 注释annotation：从头注释，同源注释，依据RNA注释
### chip-seq：研究与DNA结合的蛋白质
### ATAC-Seq：研究染色体的开放和关闭情况
### Hic：染色体在三维空间的构象
### RNAseq矩阵：横轴是样品，纵轴是基因，数值是样品落到这个基因上的reads有多少
### Microbiome矩阵：横轴是样品，纵轴是不同的微生物，数值是样品有多少来自该微生物
### RNAseq分析：在服务器上用linux进行read mapping and counting→产生一个表达表→R上分析
### 均一化CPM = count per million reads; FPKM = fragments per kilobase per million; RPKM = reads per kilobase per million;FPKM是RPKM的双端测序以read pair为单位计数的版本，per kilobase of transcripts
### 质检很重要！！！比如FastQc这个软件有一个参数percentage mapped指的是数据中百分之多少的reads能正确比对到参考序列上
### 油包水双端测序，read1主要来自barcode和UMI，有时会测到ployA和部分基因,read2主要来自基因，是从三端开始测的，所以又叫三端测序，一个珠子多个细胞的情况最不好，因为可能会导致误认为一个细胞既有T细胞的特征也有B细胞的特征

# cellranger结果图学习记录
#以下边这图为例来记录一下cellranger结果图怎么看，真正能看到的barcode比理论的barcode要多得多，因为有很多barcode没有结合到细胞，会吸附游离的RNA，捕获的RNA特别少。
#1、在cellranger运行结束之后，算法会根据每个barcode对应的umi总数的多少从大到小，给barcode排序，排名第一的barcode它对应的umi总数最多。
#2、排序之后给barcode的序号以及对应的umi总数做对数处理，x轴是barcode的排名对数，y轴是umi的数量对数，但实际上递增或递减这个现象是一致的，并没有因为取对数而发生改变，不过需要注意的是越靠近x轴左侧的barcode对应的umi数越多。
#3、理想的情况下会出现两到三个平台，第一个平台一般是质量好的细胞得到的结果。有很多时候barcode在测序时会出错，但这种情况下对应的umi数极少，会出现一个小平台。但因为前边说的实际读出的barcode会比结合到细胞的barcode多，所以第二个平台一般是空的barcode结合的环境中的RNA。
#4、另外，由于是经过对数处理的结果，在图中虽然看着第一个平台比后两个平台在x轴的方向上要宽，但实际上第三个小平台数量＞第二个平台＞第一个平台的数量，因为x轴越往右数值越大，经过指数还原后倍数更是会增加。所以实际上捕获到真实细胞并且测序未出错的才是小部分。
#Estimated Number of Cells (20,799): 该样本共捕获了约 2.1 万个 细胞。这是一个相当高的通量。
#Mean Reads per Cell (72,585): 平均每个细胞分配到约 7.2 万条 reads。这说明测序深度非常充足，通常 2-5 万条 reads 就已足够满足常规聚类需求。如果数量过少称为不饱和，一个细胞本来释放了很多核酸，如果测得reads很少，没办法体现这个细胞的核酸的多样性。Sequencing Saturation	77.5%说明了这个情况。
#Median Genes per Cell (593): 每个细胞检测到的基因中位数仅为 593 个。这是一个相对较低的数值。虽然这可能受物种（如你正在研究的七鳃鳗）参考基因组注释质量的影响，但也可能暗示细胞活性较差或测序库的多样性偏低。
#Median UMI Counts per Cell (2,027): 中位 UMI 分子数为 2,027，与低基因数相对应，说明单个细胞捕获到的转录本总量有限。一般情况下umi会比gene要多，因为umi捕获的是RNA，基因可能转录出很多RNA。
#Reads Mapped to Genome	60.3%这个值很低了，主要是因为老师做的这个参考序列和样本不是一个物种。
#Reads Mapped Confidently to Genome (40.0%): 这是一个红色信号。通常人类或小鼠样本该值应 >85%。因为一个read可能回落到基因组的很多位置上，这个指的是能确定一个read到底在哪个位置上的概率。
#Reads Mapped Confidently to Transcriptome (35.7%): 仅 35.7% 的数据落在了已知的转录本区域。
#Chemistry	Single Cell 5' PE说明是5'端测序。
#Reads Mapped Antisense to Gene	1.9%代表有 1.9% 的 Reads 虽然比对到了基因位置，但方向是反的。在链特异性文库中，这个数值通常应该很低（通常 < 5%）。如果这个比例异常高，可能意味着文库构建方向设反了，或者该物种存在大量的自然反义转录现象。
![image](https://github.com/wyyang-thy/Learning-Notes/blob/main/image.png)
#另外还需要知道一个概念叫做链特异性。在基因组中，基因的排列是非常紧密的。有时两个基因会在 DNA 的不同链上发生重叠。如果是非特异性的话，如果在这个重叠区域测到了 Read，你无法判断它属于基因 A 还是基因 B。而链特异性能明确告诉你这串序列是5'→3'还是3'→5'，从而精准地将 Read 归入正确的基因。
#链特异性文库的构建是这样的：根据RNA逆转录出一条DNA链，再根据这个DNA链合成它的互补链，不过互补链中用U替换掉T，然后将U特异性降解酶投入降解掉第二条链，这样测序时就只会测到已知方向的RNA的互补链，就可以精准的推测出RNA的序列是怎么样的。
#在测序得到的两个 Read 中，Read 1 包含 Barcode 和 UMI，而 Read 2 包含转录本的序列。Read 2 的序列方向是从 mRNA 的5'→3'方向。
#需要知道的是mRNA本身就是DNA正义链的拷贝，而且带有poly-A尾，dropseq这个技术使用poly-T去捕获RNA，逆转录得到的第一条DNA单链必然是反义链。
#Read 1 的接头（包含 Barcode 序列的一端）被固定连接在 cDNA 的 3' 端（对应 mRNA 的末端）。很容易想到，磁珠的末端依次包含barcode、UMI和poly-T，所以离barcode近的一定是3'端。是否是正义链，需要看引物的设计
#至于链特异性能辨别DNA污染，这个或许可以用分子生物学中的DNA转录的不确定性来说明，因为模板链和有义链可能在一条DNA单链上交替存在，也就是说并不是严格的某一条链就是模板链，所以如果有DNA污染的话，磁珠的poly-T尾捕获DNA扩增出来的read可能既含有正义链也有反义链序列，所以可以辨别DNA污染。
#因为PCR里边的特殊酶是热稳定DNA聚合酶，所以只能针对DNA进行扩增，而不能针对RNA进行扩增，在dropseq这个技术中，油包水这个结构形成之后，其中有逆转录酶，会在其中进行反转录，把RNA逆转录成cDNA，而且会带有barcode和umi这些信息。

# 代码部分
# 从github上安装R软件包
install.packages("remotes")
library(remotes)
remotes::install_github("drisso/mbkmeans", force = TRUE)

# 卸载软件包的方法，例如：卸载ggplot
detach(package:ggplot)

# 给矩阵或数据框增加新列的方法，mutate或summarise
data |> select(var1, var2) |> mutate(var3 = var1 + var2)# 增加第三列作为前两列的和
data |> select(var1, var2) |> group_by(var1) |> summarise(var3 = n())# 按照第一列进行分组之后统计每一组的个数

# 计算频数的函数table
table(iris$species)# 计算数据框iris的species列的频数
table(iris$species, iris$kmeans)# 输入两个变量会得到这两个变量的对应频数关系
table(iris$species, prediction)# 可以看到将species进行聚类分析的结果，进而计算分类准确率

# strsplit函数分割数值
strsplit(字符向量, "分隔符")
strsplit(iris$info, ";")# 把info这一列的以;相隔的数据分开

# grep函数筛选特定字符的位置
grep("字符", 向量名字)# 把这个向量中含有这些字符的元素筛选出来，得到的是位置，需要用向量[]取出来
grep("字符", 向量名字, value = TRUE)# 这样取出来的就是真的值而不是位置

# merge函数合并两个表格
merge(表格1, 表格2, all = TRUE)# 合并两个表格并且保留所有信息，如果没有all = TRUE只会保留两个表格相同的行名的信息

# lm函数进行线性回归
lm_test <- lm(y ~ x, data = 数据集的名字)
summary(lm_test)# 用summary查看简略信息
plot(lm_test)# 查看诊断图

# apply函数
x <- matrix(1 : 6, ncol = 3)
apply(x, 2, mean)# 2指的是第二个维度也就是列，这里是对列取平均值
y <- c()
for (i in 1 : 3)
  {y[i] <- mean(x[,i])}# 和上边apply的结果是一样的
colmeans(x)# 或者是这样也行

# PCA分析
iris_pca <- prcomp(iris[, 1:4], scale = TRUE)# pca分析
library(factoextra)
fviz_eig(iris_pca)# 可视化主成分分析
fviz_pca_ind(iris_pca, geom = "point", col.ind = iris$species, repel = TRUE)# 主成分坐标图，颜色使用species作为分类来进行标注颜色
fviz_pca(iris_pca, geom = "point", col.ind = iris$species, repel = TRUE)# 可视化样本和变量

# 聚类分析
iris_kmeans <- kmeans(iris[, 1:4], centers = 3)# 分成三个簇
iris$kmeans <- as.factor(iris_kmeans$cluster)# 把分组信息添加到iris中作为新列，这也是添加新列的好办法
fviz_pca_ind(iris_pca, geom = "point", col.ind = iris$kmeans, repel = TRUE)# 主成分坐标图，颜色使用kmeans作为分类来进行标注颜色，进行可视化聚类分析

# ggplot画图部分
ggplot(数据集，aes(x = 数据集的某一列, y = 数据集的另一列, colour = 具有分类变量的一列)) + geom_point(colour = "red") + xlab("横轴名字") + ylab("纵轴名字")# ggplot函数的基本格式：数据集，制定画图需要的列名，如果想要按照分类指定颜色的话需要加上分类变量(factor)的列名，如果指定的是一个连续变量那么颜色会渐变，加上画图的类型比如散点图，在其中可以改变点的颜色，如果前边分类变量加了颜色这里就不用加，加上横轴和纵轴的名字
violin <- ggplot(数据集，aes(x = 数据集的某一列, y = 数据集的另一列, fill = 具有分类变量的一列)) + geom_volin() + geom_boxplot(width = 0.1)# 画小提琴图的时候可以在里边画上箱型图还得指定箱子宽度，这里如果给小提琴图填充颜色使用fill
ggsave(violin, "路径")# 保存图片
ggplot(数据集，aes(x = 数据集的某一列, y = 数据集的另一列, colour = 具有分类变量的一列)) + geom_point(colour = "red") + xlab("横轴名字") + ylab("纵轴名字") + geom_smooth(method = "lm", col = "blue")# 画上一条蓝色的回归线
ggplot(数据集，aes(x = 数据集的某一列, y = 数据集的另一列, colour = 具有分类变量的一列)) + geom_point(colour = "red") + geom_smooth(method = "lm", col = "blue") + stat_regline_equation()# 将线性方程添加在图上
