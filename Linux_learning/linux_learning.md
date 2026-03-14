# 20260311_Linux学习记录

## 文件管理
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

# 把指定目录下的文件复制到另一个指定目录中去，关于cp这个命令，不管怎么样都带一个-rv的选项，一个是指定目录，一个是显示移动路径，不管干嘛就cp -rv
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
:1,5d  # 删除第1到第5行
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

:1,5 s/1/2/g  # 把1到5行的所有1换成2，g是global的意思，s是Substitute替代的意思
:% s/1/2/g  # 把整个文件的所有1换成2
:1,5 s/1/2/  # 把1到5行的第一个1换成2
:w ./2.txt  # 把当前的内容另存为2.txt，原来的1.txt仍然存在
:%d  # 清空整个文件的内容，%代表整个文件
```

## 用户管理
```
root:x:0:0:root:/root:/bin/bash  # 用户名：密码占位符：用户id：组id：描述：家目录，登陆时所在的目录：登录shell，命令解释器（定义、接收、解释命令）

# 添加一个新的用户test01
useradd test01
# 因为普通用户的id是从1000开始命名的，所以在添加新的用户的时候还可以指定它的用户id
useradd test02 -u 1530
# 指定home目录
useradd test03 -d /home/test03
# 查看用户id和组id等信息
id test01
# 修改密码
passwd test01
# 切换到另一个用户
su - test01
# 查询用户
whoami
# 删除用户
userdel -r test01  # -r是家目录一起删掉
# 修改用户属性
usermod -s /sbin/nologin user01   # -s的意思是shell
# 查看修改
tail -1 /etc/passwd  # passwd和shadow存储的都是用户信息，而不是组信息，但是会存有基本组的gid，所以下边的group信息不在这里，输出user01:x:1000:1000::/home/user01:/sbin/nologin，现在这个用户没办法登录shell了

# 添加hr组
groupadd hr
# 查找信息
cat /etc/group
grep hr /etc/group  # 查询相关信息
# 删除hr组
groupdel hr
# 组分基本组和附加组，这是相对于用户来说的，基本组会随用户的创建而随之建立且名字和用户一致，比如张三如果加入李四的组中，那么张三会得到一个名为李四的附加组，张三还有一个名为张三的基本组。并且一个用户可以加入多个组，也就是说可以有多个附加组。
# -g设定用户的基本组，-G设定用户的附加组也就是加入另外一个用户的分组
# /etc/passwd可以查看用户的基本组，/etc/group可以查看用户的附加组，id也可查看附加组

useradd AAA
useradd BBB
useradd CCC
grep AAA /etc/passwd  # AAA:x:1000:1000::/home/AAA:/bin/sh
grep BBB /etc/passwd  # BBB:x:1001:1001::/home/BBB:/bin/sh
grep CCC /etc/passwd  # CCC:x:1002:1002::/home/CCC:/bin/sh
# 修改AAA的基本组为CCC
usermod AAA -g CCC
grep AAA /etc/passwd  # AAA:x:1000:1002::/home/AAA:/bin/sh
# 修改CCC为BBB的附加组
usermod BBB -G CCC
grep BBB /etc/passwd  # BBB:x:1001:1001::/home/BBB:/bin/sh
grep BBB /etc/group  # BBB:x:1001:; CCC:x:1002:BBB（这个意思是CCC的基本组是BBB也就是说是BBB加入了CCC组）
# 现在CCC组中有AAA和BBB两个新成员
id AAA  # uid=1000(AAA) gid=1002(CCC) groups=1002(CCC)
id BBB  # uid=1001(BBB) gid=1001(BBB) groups=1001(BBB),1002(CCC)
# 把组员A从组中删除gpasswd -d A GROUP，但是不能从基本组中删除，只能从附加组中删除
gpasswd -d BBB CCC  # Removing user BBB from group CCC
id BBB  # uid=1001(BBB) gid=1001(BBB) groups=1001(BBB)

groupadd DDD
grep DDD /etc/group  # DDD:x:1003:
groupmod -g 1004 DDD
grep DDD /etc/group  # DDD:x:1004:
```
