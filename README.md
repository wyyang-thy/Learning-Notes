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
