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

# 以下的命令都是在root用户下进行的，如果是普通用户都得加一个sudo
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

useradd AAA  # useradd AAA -G hr  或者建立的时候就直接给一个附加组
useradd BBB
useradd CCC
grep AAA /etc/passwd  # AAA:x:1000:1000::/home/AAA:/bin/sh
grep BBB /etc/passwd  # BBB:x:1001:1001::/home/BBB:/bin/sh
grep CCC /etc/passwd  # CCC:x:1002:1002::/home/CCC:/bin/sh
# 修改AAA的基本组为CCC
usermod AAA -g CCC
grep AAA /etc/passwd  # AAA:x:1000:1002::/home/AAA:/bin/sh
# 修改CCC为BBB的附加组，也就是BBB加入CCC组
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

## 用户的权限
```
# 授权r=4、w=2、x=1
chmod u+r 1.txt  # 给这个文件的用户读的权限
chmod g-r 1.txt  # 不给这个文件的同组用户读的权限
chmod u=rwx 1.txt  # 给这个文件的用户读、写、执行的权限
chmod -R u+r ./aa/  # 给这个文件夹的用户读的权限
chmod a=rx 1.txt  # a是all包括user、group和other，这是给所有人rx权限
chmod +x file1  # 不指定就所有都加x
chmod ug=rx,o=r 1.txt  # 相当于chmod 554 1.txt
# 如果要看文件夹./workspace的权限要加-d
ls -lhd ./workspace

vim file1
# 在文件中写入这些话
echo "hello"
read -p "please input your name:" name  # read核心命令。等用户输入一点东西，按回车结束。-p参数 (Prompt)。意思是“提示”。如果没有它，屏幕会一片漆黑，用户不知道该干嘛。name变量名。用户输入的内容会被“装”进这个叫 name 的变量里。
echo "haha $name is a fool"
# 给权限并执行文件
chmod +x file1
bash file1  # 或者直接./file1也是运行

# 对于文件夹也同样适用，但是文件夹必须有执行权限不然进不去，对文件夹的一般操作不会影响文件夹下的权限，但是如果对于文件夹的操作带了递归选项-R文件夹包括文件夹下的所有文件权限都会改变!!!!!!!
# 修改一个文件的组：chown 用户名.组名 文件名
groupadd hr
useradd user0  # 每个用户都有自己的基本组，所以user0也有一个名为user0的基本组
chown user0.hr 1.txt  # -rwxrwxrwx 1 user0 hr    936 Mar 13 15:04 1.txt
chown user0 1.txt  # 只改用户
chown .hr 1.txt  # 只改组
# chgrp只能修改所属组
chgrp user0 1.txt

# 但是如果三个用户以及两个组对这个文件的权限分别都不同，上边的命令就不能满足了，因为上边的是只能对一个用户或一个组进行操作，这时候需要ACL(access control list访问控制列表)来定义谁可以做什么
setfacl -m g:hr:rwx 1.txt  # 设置文件访问控制列表 -设置make 组：组名：权限 文件名
setfacl -m o::rwx 1.txt  # 这个是给文件的other其他用户加权限
getfacl 1.txt  # 查看文件有什么acl权限
# 输出显示：
  # file: 1.txt
  # owner: user0
  # group: user0
  user::rwx
  group::rwx
  other::rwx

setfacl -x g:hr 1.txt  # 删除hr的acl权限，另外如果使用acl对于文件进行权限操作的话-rwxrwxrwx+会有一个+显示
setfacl -b 1.txt  # 删除所有的acl权限
setfacl -d 1.txt  # 回到设置acl之前的样子

# suid是针对文件所设置的一个特殊的权限，使得调用文件的用户临时具备属主的能力，但是这个权限需要谨慎使用
# 假如在root下有一个文件file1.txt，那么除了用户属主，组中的用户以及其他用户是没有查看这个文件的权限的
chmod u+s /root/file1.txt  # 如果给这个文件加上一个suid的权限，那么所有的用户都能查看这个文件，这个文件也将会变成-rwsr-xr-x，注意user的权限是rws，注意这里是小写s，是因为之前这个位置有x，如果没有x那么将会变成S大写的S

# i权限赋予一个文件连超管也无法更改、重命名、删除的能力
chattr +i file1.txt
lsattr file1.txt  # 列出属性
chattr -i file1.txt  # 去掉这个属性
chattr +a file1.txt  # 增加一个只能追加的权限，vim不行，只能追加
echo 111 >> file1.txt  # 在屏幕上能打印的东西都能追加进文件，echo相当于print

# umask进程掩码，会对新建立的文件和文件夹产生影响，用于去除权限，创建新文件夹的时候是用0777-0022也就是0755所以新建的文件权限都是rwxr-xr-x，创建新文件的时候是0755-0111也就是644也就是去掉了新文件所有执行的权限
umask  # 0022，掩码是四位数，之前chmod的时候修改都是后三位，第一位是给suid权限，比如4777那么就是rwsrwxrwx，2777就是rwxrwsrwx，1777就是rwxrwxrwt，0777就是rwxrwxrwx
chmod 4777 file1.txt
```

## 进程管理，进程是正在运行中的程序。每个任务在运行时都会有一个process id也就是进程号
```
# 静态查看进程ps(process status进程状态管理器)
which ps  # 查询这个命令的路径
# a：显示所有用户（all）在所有终端上运行的进程。u：以用户为主（user-oriented）的格式输出，会显示进程的所有者、内存和 CPU 占用等详细信息。x：显示没有控制终端的进程。这通常包括许多在后台运行的系统守护进程（Daemons）。
ps aux | head -3  # r是running运行、t是stop暂停、s是sleeping休眠、z是zombie僵尸进程exit后还占用内存，此时靠pid杀死进程kill
ps aux --sort -%cpu  # 按照占用cpu多少排序输出，其中-%cpu是降序排序，没有-就是升序
ps aux --sort -%mem  # 按占用内存降序排序
ps -ef  # 可以直接查询pid和ppid(parent process id)
# o (Output)：这是关键参数。它告诉 ps：“不要用默认格式，按照我指定的列来显示。”
ps axo user,pid,ppid,%mem,command  # 自定义只看user,pid,ppid,%mem,command这几个

# 动态查看进程top，按q退出，按h是help展示所有的命令，P和M分别是按降序展示cpu和内存占用，其中的NI是nice的意思，代表优先级， VIRT、RES、SHR都是内存
top -d 3  # 默认是1秒刷新，现在是3秒刷新
top -d 3 -p 1  # 查看进程号是1的进程
top -d 3 -p 1，300  # 查看进程号是1和300的进程

ps aux | grep 1.txt  # 用ps查看pid，为2366
top -d 3 -p 2366  # 用top查看占用

kill -l  # 会列出所有kill支持的选项
kill -1 8848  # 重新加载进程号8848的配置
kill -2 8848  # 中断8848
kill -9 8848  # 强制终止
kill -15 8848  # 正常关闭进程，也是默认的选项相当于kill 8848

free -m  # 以兆为单位看内存，也可以以k为单位看
df -hT  # 看磁盘大小空间

# nice可以设置进程的优先级，优先级一共从-20到19有40个数字，nice值越大优先级越低，19最低，-20最高，这个是手动能调节的范围，系统也有一个优先级，我们设置的数字传给系统之后会+20再传给系统
# 默认进程会继承父进程的优先级，一般默认是0也就是居中
ps axo pid,command,nice --sort=nice  # 这个会按优先级从大到小排序，虽然是降序，但是nice值越小优先级越大
sleep 2000 &  # &这个符号会让sleep 2000命令到后台进行
nice -n -5 sleep 2000 &  # 设置优先级为-5
ps axo pid,command,nice | grep sleep  # 查看优先级
# 给已经有优先级的进程重新赋值优先级
sleep 3000 &  # 3615
ps axo pid,command,nice | grep sleep  # 优先级为0
renice -20 3615  # 3615 (process ID) old priority 0, new priority -20
ps axo pid,command,nice | grep sleep  # 优先级为-20
top -d 3 -p 3615  # 也可以用top命令查看进程的优先级，也能看到名为PR也就是prior，是系统的优先级，会是nice值+20
```

## 作业管理
```# 通过&把任务放到后台
sleep 2000 &
jobs  # 查看后台运行程序，第一列是作业号
fg 1  # 把1号作业调回前台也就是foreground，ctrl z会把作业暂停之后丢到后台
bg 1  # 把在后台停止运行的作业继续运行，也就是background
kill %1  # 消灭后台作业号为1的作业，后台就没有这个作业了
```

## 虚拟文件系统
```
cat /proc/cpuinfo  # 看硬件信息
cat /proc/meminfo  # 看内存信息
cat /proc/cmdline  # 看内核信息
```

## 管道和重定向
```
# 重定向>是覆盖，>>是追加，只有能够在终端页面上能够输出的东西才能够使用重定向！！！这两个命令相当于1>和1>>只会把正确信息放到后边的文件中，同理2>和2>>只会输出错误的信息到文件中
ls /aaaa &> /dev/null  # 这是这条命令最常用的地方，&意思是1和2也就是正确错误都输出到null中，也就是垃圾桶中，不在终端显示任何信息
ls /aaaa > 1.txt 2> 2.txt  # 把正确和错误分开放
# 输入重定向<
vim word.txt  # 在这个文件中写入一封邮件
mail -s "test01" alice < word.txt  # -s参数代表Subject（主题）。这里设置的主题是 "test01"。把文件中的内容当作邮件发送，否则会在终端上输入很多文字
su - alice  # 切换到alice用户
mail  # 查看邮件

# pipe管道,|是管道符，也就是会把左边命令的输出作为右边命令的输入
cat /etc/passwd | tail -3 | grep 'user'
# tee管道除了pipe的作用还会增加另外一个方向，会把一个命令输出的结果存到另外一个文档中
cat /etc/passwd | tee file88.txt | tail -3 | grep 'user'  # 该有的|还是得有，只不过在中间加一个tee命令存上一步的输出

# cp和rm是不能用在管道中的，这是这两个命令的规则，但是如果在命令前加上xargs进行参数传递那么就可以使用
mkdir file1 file2 file3  # 在当前目录下建立这三个文件
echo /root/file1 >> test.txt
echo /root/file2 >> test.txt  # 把这三个文件的路径全部写入文件中
echo /root/file3 >> test.txt
cat test.txt | xargs rm -rvf  # 删除这三个文件
```

## 磁盘管理
```

# 软连接
ln -s 2.txt 333  # 把文件2.txt连接到333中去，但是文件删除333将无法访问
```
