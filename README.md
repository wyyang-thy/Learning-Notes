# Learning-Notes
Record some of my learning processes.
# 我的学习记录
#2026.01.29
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


# 画图部分
# ggplot函数的基本格式：数据集，制定画图需要的列名，如果想要按照分类指定颜色的话需要加上分类变量(factor)的列名，如果指定的是一个连续变量那么颜色会渐变，
# 加上画图的类型比如散点图，在其中可以改变点的颜色，如果前边分类变量加了颜色这里就不用加，加上横轴和纵轴的名字
ggplot(数据集，aes(x = 数据集的某一列, y = 数据集的另一列, colour = 具有分类变量的一列)) + geom_point(colour = "red") + xlab("横轴名字") + ylab("纵轴名字")
violin <- ggplot(数据集，aes(x = 数据集的某一列, y = 数据集的另一列, fill = 具有分类变量的一列)) + geom_volin() + geom_boxplot(width = 0.1)# 画小提琴图的时候可以在里边画上箱型图还得指定箱子宽度，这里如果给小提琴图填充颜色使用fill
ggsave(violin, "路径")# 保存图片
