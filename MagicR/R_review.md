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
