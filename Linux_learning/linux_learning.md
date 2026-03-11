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
```

## 用户的权限
