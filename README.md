# Learning-Notes
Record some of my learning processes.
# 我的学习记录

# 基础部分
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
