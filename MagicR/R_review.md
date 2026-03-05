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

### 列表是一个多维向量
```
a <- 1:10
b <- c("apple", "banana", "cherry")
c <- list(name = "John", age = 30, city = "New York")
d <- data.frame(id = 1:5, value = c(10, 20, 30, 40, 50))# 数据框定义的时候可以在前边指定列名比如id和value
alist <- list(A = a, B = b, C = c, D = d)# 在形成列表时也可以给每一个元素设置名字
alist[1]# 取出的是一个列表，是list
alist[[1]]# 取出的是元素本身，是numric
```

### 处理缺失值
```
library(VIM)
sleep
# 第一种方法，删除含有缺失值的行
omit_sleep <- na.omit(sleep)
# 第二种方法，使用均值填充缺失值
mean_sleep <- sleep
mean_sleep$extra[is.na(mean_sleep$extra)] <- mean(mean_sleep$extra, na.rm = TRUE)# 使用这一列中非缺失值的均值填补这一列中的缺失值
# KNN填充缺失值
knn_sleep <- kNN(sleep, variable = "extra", k = 5)# extra这一列中用knn进行插值
```

### 排序的两种方式，sort()是直接对于数值进行排序，order()对于索引进行排序（其实也是对数值进行排序，展示的是最小的数值对应的索引到最大的数值对应的索引），在二维数据中行排序常用order函数

### subset()函数筛选数据，注意判断相等时使用==
```
# subset(x(数据框), subset(行筛选条件（逻辑表达式，也就是某一列的判断，比如dose列>5), select(列选择（指定保留哪些列）))，行筛选条件指的是用列的值来筛选满足条件的行
a_mtcars <- subset(mtcars, cyl ==6)
a_mtcars <- mtcars[mtcars$cyl ==6, ]
```

### sample(向量，抽样数目，replace = T)函数随机抽样，replace代表是否有放回的抽样
```
x <- 1 : 10
sample(x, size = 5)# 一维
# 扑克牌
# 生成一副扑克牌
# 花色
type <- c("red", "spades", "cube", "plum")
# 数值
amount <- c("A", 1 : 10, "J", "Q", "K")
poker <- paste(rep(type, each = 13), amount, sep = "")
length(poker)
poker[c(53, 54)] <- c("red joker", "black joker")
poker
# 洗牌
set.seed(1234)
shuffle <- sample(poker, 54, replace = FALSE)# 一维，不放回的随机抽54次就相当于洗牌
# 底牌
dipai <- shuffle[c(52, 53, 54)]
# 分牌
one <- shuffle[1 : 51][c(T, F, F)]
two <- shuffle[1 : 51][c(F, T, F)]
three <- shuffle[1 : 51][c(F, F, T)]
one
shuffle

# 二维数据随机抽样，需要先计算行数，根据行数随机抽样得到行索引，根据行索引得到该行
sleep[sample(1:nrow(sleep), size = 5, replace = FALSE), ]

# 划分训练集和测试集，7：3的比例
set.seed(123) # 设置随机种子以确保结果可重复
train_index <- sample(1:nrow(sleep), size = 0.7 * nrow(sleep), replace = FALSE)
train_data <- sleep[train_index, ]
test_data <- sleep[-train_index, ]# 直接-索引，代表去掉这些索引
```
## tidyverse以及dplyr的使用
### tibble也就是原来基础R里的数据框，但它不支持行名，而且它取出一列还是tibble
```
library(tidyverse)
library(dplyr)

# slice()函数筛选行
iris <- iris |> slice(1 : 3)# 筛选前三行

# select()函数筛选行
iris <- iris |> select(1 : 3)# 筛选前三列
iris <- iris |> select_all(toupper)# 把所有行都转换成大写字母
iris <- iris |> select_at(vars(-contains ("ar"), starts_with("c")),toupper)# 把不包括ar的列并且以c开头的列换成大写

# pull()函数筛选行但是返回向量
iris <- iris |> pull(1 : 3)# 筛选前三列

# add_row()和add_column()函数增加行列
df |> add_column(new = 1:10)# 新列的名字是newid

# mutate()函数比较重要啊！！！也是增加一列
mtcars %>% mutate(cyl2 = cyl * 2)# cyl2是新列的名字

# rename()函数修改列名
mtcars %>% rename(cyl2 = cyl)

# arrange()函数默认升序，还可以使用两列进行排序
mtcars  %>% arrange(desc(cyl, mpg))

# 去除重复，basrR中是unique()函数，现在是distinct()函数
mtcars %>% distinct(cyl, .keep_all = TRUE)# 只对这一列处理，其他列保留

#filter()函数，默认是and关系，对列的值进行过滤筛选，select函数是根据列名进行筛选
mtcars %>% filter(cyl == 4 | cyl == 6)  %>% relocate(mpg, .after = cyl)# 把mpg放在cyl这一列的后边
mtcars %>% filter(duplicated(x, y))# 把xy值相等的行保留下来

# summarise()函数相当于summary()函数汇总数据
mtcars %>% summarise(mean(mpg))

# group_by()函数进行分组，但是必须是因子
mtcars %>% group_by(cyl)

# count()函数相当于table()函数计算频数
mtcars %>% group_by(cyl) %>% count()

# left_join(x,y)将y并入x，right_join(x,y)将x并入y，inner_join()保留相同的行，full_join()全部保留，intersection()交集，union()并集

# sample_n根据数量进行抽样，sample_frac()根据比例进行抽样

# tidyverse中的stringr这个包是专门用于处理字符串的包
```
