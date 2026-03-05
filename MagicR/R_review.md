# 在复习R的过程中记忆深刻的知识点
## 在R的使用中少使用循环，多使用向量化操作
## 画图的原则：先让信息展现出来，再根据信息画别的图，比如先用plot()函数展现出所有的点

### ifelse(判断条件, yes = , no = )，在满足判断条件的值会被替换成yes后的值，不满足条件则替换成no后的值，不会在原向量中进行更改
```
data(rivers)
x <- ifelse(rivers > 500, yes = 1, no = 0)
```

### 对于数值向量，range()函数可以取到它的范围也就是最小值到最大值
```
range(rivers)
```

### round(x, digits = 2)函数对于x四舍五入保留2位小数
```
round(mean(rivers), digits = 2)
```

### 两个向量的T检验
```
# 随机生成两个向量
x <- runif(100, min = 1, max = 100)
y <- runif(100, min = 2, max = 200)
t.test(x, y)
```

### 对向量进行快速统计的函数summary()函数，可以快速找到最小值、最大值和四分位数
```
summary(rivers)
```

### plot()函数画散点图，可以根据pch参数选择不同的点形状，一共18种类型，颜色使用col参数，类型type是画的类型p是点，l是线，b是点和线都画，xlim和ylim是决定从什么坐标开始画图
```
plot(0 : 18, pch = c(0 : 18), col = "red", type = "p", xlab = "横轴", ylab = "纵轴", main = "主标题", sub = "子标题", xlim = c(1, 100), ylim = c(0, 100))
```

### attach()函数可以将数据框中的每一列单独取出变成向量，和数据框$列名的作用一致
```
data(women)# 数据框
women$height
attach(women)
height# 可以单独作为向量取出
plot(x = weight, y = height, pch = 19, col = "red", type = "b")
```
### 条形图的上下左右的距离可以通过par函数来进行调整，mar = c(下x标签, 左y标签, 上主标题, 右)
```
barplot(state.area, names.arg = state.abb, las = 2, col = "blue")
par("mar")

# 增大底部边距，防止标签被截断
par(mar = c(8, 4.1, 4.1, 2.1))

# 其中density指的是条形图中的斜线数量，angle是斜线的角度，horiz默认是FALSE也就是竖着展示，也可以改成横着展示，也需要通过mar来调整边界
barplot(state.area, names.arg = state.abb, las = 2, col = "blue", density = 15, angle = 45, horiz = FALSE)
```

### 非常常用的一个函数就是table()函数，来计算频数，计算频数之后还可以计算卡方检验
```
data(Arthris)
chisq.test(table(Improved, Sex))
```

### dim()函数给一个一维向量增加维度可以变成高维
```
x <- c(1 : 100)
dim(x) <- c(10, 10)
```

### R语言中的向量化操作，使用c(T, F)可以交替的取出奇数位置的数值
```
data(state.x77)
state.x77[c(T, F), ]
```

### cor()函数计算两变量值之间的相关性
```
# 这样会得到一个关于主对角线对称的矩阵，计算的是变量值也就是每两列之间的相似度
round(cor(state.x77), digits = 2)
```
### corrplot包绘制相关性图
```
library(corrplot)
m <- cor(mtcars)# 计算相关性结果
corrplot(m, method = "circle")# 用圆圈代替相关性
corrplot(m, method = "number")# 相关性结果

# 带斜线阴影的色块，正相关和负相关用不同方向的斜线区分。"hclust" 表示使用层次聚类（hierarchical clustering） 对变量重新排序，将相关性强的变量排在一起，便于发现变量间的分组模式。FALSE 表示隐藏对角线，
corrplot(m, method = "shade", order = "hclust", diag = FALSE)
```

### 中心化标准化数值-zscore数值scale函数
```
x <- as.matrix(mtcars)
heatmap(x)
y <- scale(x)
heatmap(y)
# pheatmap画图更好看
library(pheatmap)
pheatmap(y)
```

### 箱线图和小提琴图，连续数据（数值）~离散变量（分类）
```
boxplot(len ~ supp, data = ToothGrowth)
boxplot(ToothGrowth$len ~ ToothGrowth$supp)

library(vioplot)
vioplot(len ~ supp, data = ToothGrowth, col = "lightblue")
```

### apply(二维数据， 1（行）/2（列）， 函数)系列函数，也就是对这个二维数据的行或列进行运用什么函数
```
apply(X = state.x77, MARGIN = 2, FUN = mean)# 需要注意的是里边的参数名称都是大写字母，这里的效果相当于colMeans()函数

# sapply()函数对于一维数据进行处理，不需要指定行或列，返回向量
# lapply()对于列表进行处理，不需要指定行或列，返回列表
lapply(state.center, mean)
sapply(state.center, mean)

# tapply()对于因子进行处理，第一个参数写数值变量，第二个参数写分类也就是因子变量，用来计算每一个分类变量下所有数值变量的对应操作
tapply(mtcars$mpg, mtcars$cyl, mean)
library(vcd)
Arthritis
tapply(Arthritis$Improved, Arthritis$Treatment, table)# 对于两个都是分类变量就不能用数值运算，可以用table等计量运算
tapply(Arthritis$Age, Arthritis$Treatment, mean)
```
