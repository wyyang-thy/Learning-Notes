#### ONT（Oxford Nanopore Technologies）是牛津纳米孔单链测序：想象一张薄膜上有一个极其微小的洞，这个洞就叫“纳米孔”。科学家把 DNA 双链解开，变成单链（就像把拉链拉开，只拿其中一半）。然后，像穿针引线一样，让这条单链 DNA 从这个极小的纳米孔里钻过去。在这个孔里，一直有微弱的电流通过。因为 DNA 上有四种不同的碱基（A、T、C、G），它们的形状和大小微有不同。当它们依次挤过纳米孔时，会像“路障”一样不同程度地阻挡电流，产生独特的电流变化。机器只要记录下这些电流的变化，就能反推并“读”出刚刚钻过去的是什么基因。ONT 最大的优势就是极其长。它可以一口气读出几十万甚至上百万个碱基组成的超长片段。如果说以前的测序是把文章撕成一个个单词来拼，ONT 就是直接给你一整页、甚至一整章的内容。不够准确但够长。
#### PacBio HiFi 测序。HiFi 是“High Fidelity”（高保真）的缩写。科学家先把一段较长的 DNA 片段的两头接上像发夹一样的接头，把它变成一个环状（就像把一根细细的鞋带两头系在一起，变成一个手镯）。然后，测序机器像一辆沿着环形跑道开的赛车，开始在这个“手镯”上一圈又一圈地反复读取同一段 DNA。任何机器读取时都会犯错。但因为机器在环形 DNA 上读了一圈又一圈，虽然每一圈都可能在不同的地方读错，但只要把这几十圈的结果叠在一起互相校对（少数服从多数），那些随机的错误就被抵消了。最终，它能输出一个准确率高达 99.9% 的长片段。虽然读出来的片段很长（大约1万-2万个碱基），但相比于 ONT 动辄上百万的“超长读长”，HiFi 还是偏短了一点。所以HIFI没有办法把高度抽工夫的区域拼接起来，在HIFI的结果中只能看到无数条一模一样的reads，但并不知道应该拼接在哪里，这时候需要ONT来指导框架。够准确但不够长。
#### 在完成首个 100% 人类基因组（T2T）时，科学家把这两项技术结合在了一起：HiFi 负责“精装修”： 因为它极度准确，科学家先用 HiFi 测序生成了大量极其精确的基因组区块。它保证了这幅拼图的每一个细节都完美无瑕。ONT 负责“搭骨架”： 当 HiFi 遇到那些实在太长、跨越不过去的“高度重复区域”时，ONT 就出场了。利用它无与伦比的“超长读长”，ONT 就像搭建了一座长桥，直接跨越了盲区，告诉科学家这片巨大的重复区域整体结构是怎样的。难怪大连给我们的T2T原始数据既有ONT也有HIFI...

#### 另外，我尝试使用opencode
```
curl -fsSL https://opencode.ai/install | bash
source ~/.bashrc
opencode --version
opencode
# ctrl c可以退出
/connect连接了deepseek V4 pro 速度为max的大模型
```
#### 从pi节点向冷存储传送数据记录
```
# 先登录pi的数据传送节点，一般登录的是登录节点ssh wyyang2025@pilogin.hpc.sjtu.edu.cn但是大量传送数据的时候会被系统kill掉。
ssh wyyang2025@data.hpc.sjtu.edu.cn

# 使用tmux会话传送，开启一个名字为transport的会话
tmux new -s transport

# 使用rsync传送，这个兼顾了软硬链接，把结果传送到了union下的workspace节点下/union/home/acct-medcl/wyyang2025/workspace/，得先建立一个文件夹workspace才能这样
rsync -avrPH --copy-unsafe-links /lustre/home/acct-medcl/wyyang2025/workspace/Accuramed20260228 $UNION/workspace/

# 可以在data节点cd到union路径下进行修改文件夹什么的命令，但是不能在登陆节点有这些操作
# ctrl b撒开按d就可以离开传送会话窗口

# 回到名为 transport 会话
tmux attach -t transport

# 彻底关闭 transport 会话
tmux kill-session -t transport
```
#### 另外，尝试从本地把数据传送到超算冷存储上
```
# 在 WSL 中创建一个挂载点（文件夹）
sudo mkdir -p /mnt/f
# 将 F 盘挂载到这个目录
sudo mount -t drvfs F: /mnt/f
# 这样我就能用cd /mnt/f访问这个移动硬盘了

# 在拔出物理硬盘前，记得先在 WSL 里解除挂载：
sudo umount -l /mnt/f

# 开一个隐藏入口
screen -dmS transt2t
screen -r transt2t
# 直接传送到冷存储下
scp -r /mnt/f/T2T_rawdata wyyang2025@data.hpc.sjtu.edu.cn:/union/home/acct-medcl/wyyang2025/workspace/
# 为了支持断点续传，我还是用了rsync
rsync -avrPH --copy-unsafe-links /mnt/f/T2T_rawdata wyyang2025@data.hpc.sjtu.edu.cn:/union/home/acct-medcl/wyyang2025/workspace/
# ctrl a d一起按就退出隐藏窗口

# 但是我不可能一直开着电脑，所以我决定用实验室的服务器传一下试试
# 服务器的/mnt下有一个文件夹就是挂在硬盘的我就不新建了，结果是我没有权限，让师兄帮我挂载在tmpdata4上边，tmpdata3里边存储的是20260228-Accuramed，可以用这个直接传到硬盘里了
sudo mount -t drvfs F:/mnt/portable
# 依旧是开一个screen
rsync -avrPH --copy-unsafe-links /tmpdata4/T2T_rawdata wyyang2025@data.hpc.sjtu.edu.cn:/union/home/acct-medcl/wyyang2025/workspace/
```
#### 我尝试把医院服务器上的T2T建立软链接到我的wsl上使用claude code进行拼装基因组，需要注意的是仍然需要连接服务器Wifi才能访问软链接而且每次连接网络的时候都需要重新挂
```
# 安装 sshfs
sudo apt install sshfs

# 创建挂载点
mkdir -p ~/mnt/server

# 挂载远程目录
sshfs YangWenyan@192.168.8.4:/tmpdata3/T2T_rawdata ~/mnt/server

# 建立软链接
ln -s ~/mnt/server /home/wyyang/workspace/T2T
# lrwxrwxrwx 1 wyyang wyyang 23 Jun 12 11:51 server -> /home/wyyang/mnt/server,不过后来我给server改名成T2Tonrenji了
```
#### claude code小tips
##### 原来在不同的目录下打开claude那次的会话就会存入相应的目录下，/resume的时候是找不到在其他目录的会话的
##### cd ~/workspace/T2T比如在这个目录下打开的calude这次会话只在这个目录下才能找到
```
vim ~/.bashrc
alias opencode='/home/vin/.opencode/bin/opencode'# 因为我没有安装opencode权限所以我只能把这个目录改个名字存在我这
source ~/.bashrc# 刷新起效
```
