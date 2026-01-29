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
