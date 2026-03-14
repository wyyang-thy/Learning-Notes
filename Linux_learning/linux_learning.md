# 20260311_Linux学习记录

## 文件和用户管理
### 一般都是这样的格式：命令 选项 参数
```
# 转到根目录下
cd /

# 列出./workspace下的所有文件信息
ls -lh ./workspace  # 选项中的l指的是长格式也就是详细信息，h代表有单位表示文件大小

# 创建一个文件var1
touch ./var1

# 创建一个嵌套文件夹/aa/bb/cc
mkdir -p /aa/bb/cc  # 其中的p是parent的意思，也就是创建父级目录

# 把指定目录下的文件复制到另一个指定目录中去
cp ./aa/bb/cc/test.txt ./aa/bb
# 把指定目录复制到另一个目录中
cp -r ./aa/bb/cc ./aa/  # 其中r是递归recursion的意思，涉及到目录的操作需要递归

# 移动命令，移动命令即使是移动文件夹也不需要加选项，直接移动就好
mv ./aa/cc ./
mv ./aa/cc ./ccnew  # 起一个新的名字，就是移动的时候改名字，不改路径就是直接改名字

# 删除命令rm，选项-rf中r是文件夹递归recursion的意思，f是强制force的意思
rm -rf ./aa/*  # *是通配符，代表任意东西，这里是删除aa文件夹的所有东西
rm -rf ./aa//d*  # *是通配符，代表任意东西，这里是删除aa文件夹的所有以d开头的东西
rm -rf ./aa/a1 ./aa/b2  # 同时删除aa下的a1和b2两个文件

# 查看文件的命令
cat -n chakan.txt  # 在屏幕上打印出所有的内容，-n的意思是显示行号
head chakan.txt -n 2  # 看文件的前2行
tail chakan.txt  # 查看文件的后10行，不指定-n就是默认10行
more chakan.txt  # 按enter回车有翻行的功能，空格翻页
less -NRS chakan.txt  # 其中N是显示行号，R是保留颜色，S是不自动换行方便查看原行

# 过滤关键字命令初识
grep dd ./chakan.txt  # 在这个文件中找到所有含有dd的一行

# 重定向符号>和>>
ls -lh > 1.txt  # 把输出的命令覆盖到1.txt中
ls -lh >> 1.txt  # 把输出的命令追加到1.txt中

# gedit命令，也就是graphic edit是图形化界面，以记事本的形式打开一个文件进行操作
gedit 1.txt

# vim命令，下边的所有命令都是在命令行模式下进行的，不能在插入编辑模式下使用
# 按v键进入可视化模式可以选择字符，V可以选择行，ctrl v可以选择块，按下一个y是复制，一个d是删除，一个p是粘贴
yy  # 复制
2yy  # 复制光标所在的行开始的两行
p  # 粘贴
dd  # 删除
2dd  # 删除光标所开始的两行
D  # 删除从光标位置到行尾
u  # 撤销undo上一步操作
ctrl r  # 重做上一次的撤销操作

gg  # 到达页首
G  # 到达页尾
3G  # 到达第三行
0  # 到达行首，相当于home
$  # 到达行尾，相当于end

:set nu  # 显示行号
:set nonu  # 不显示行号
:set hlsearch  # 查询高亮
/string  # 查询字符串
n  # 查询的字符向下移动
N  # 查询的字符向上移动
:set list  #显示控制字符

:1,5 s/1/2/g  # 把1到5行的所有1换成2，g是global的意思，s是switch的意思
:1,5 s/1/2/  # 把1到5行的第一个1换成2
:w ./2.txt  # 把当前的内容另存为2.txt，原来的1.txt仍然存在
```

## 用户的权限
