# awk的用法：awk '条件 {操作}' 文件名
## 条件是在单引号之内花括号之外的，但是操作比如print是必须在花括号之内的
```
awk '{print $1}' file           # 每行打印第1列
awk '$3=="exon"' file           # 只打印第3列是 exon 的行，如果只有一个condition默认的操作是print
awk 'NR>1' file                 # 跳过第一行（表头）
awk '{s+=$2} END{print s}' file # 第2列求和，最后输出
```
# 可供选择的模块有BEGIN和END，也就是开头如何操作和结尾如何操作，如果只有BEIGIN/END而没有BODY的话只会输出对应的头和尾部信息
```
awk 'BEIGIN{printf "sr no\tname\tsub\tmarks\n"}' marks.txt
# 输出的是sr no   name    sub     marks
# 这里需要注意的是printf只会输出一行不会自动换行，得手动加上\n来进行换行\t是制表符用来对齐txt中的每一列的
```
```
# AWK在BEGIN语句中打印出标题。然后再BODY语句中，它会读取文件的每一行然后执行AWK的print命令将每一行的内容打印到标准输出。这个过程会一直重复直到文件的结尾。
awk 'BEIGIN{printf "sr no\tname\tsub\tmarks\n"} {print}' marks.txt
# sr no   name    sub     marks
# 1)  Amit    Physics  80
# 2)  Rahul   Maths    90
# 3)  Shyam   Biology  87
# 4)  Kedar   English  85
# 5)  Hari    History  89
```
# awk的body的操作部分，如果把几个挨着写就是把它们连成一个字符串
```
# $3   +   "\t"   +   $4
# 第3列     Tab      第4列
# 至于\t为什么要放在双引号中，是因为awk中字符串都得放在双引号中否则awk不认识，这样的话就是对这个文件的每一行都执行输出第三列 制表符 第四列
awk '{print $3 "\t" $4}' marks.txt
# Physics 80
# Maths   90
# Biology 87
# English 85
# History 89
```
# awk中的查找命令
```
awk '/a/ {print $0}' marks.txt
# $0的意思是按照原来的格式输出，查找所有包含字母a的行和列都输出
awk '/a/ {++n} END{print "count=" n}' marks.txt# 这个是每查找到一个n就+1，然后最后输出count=n
awk 'length($0)>18 {print}' mask.txt# 这个是输出长度大于18的行，$0好像就是一整行/当前行的意思。
```


