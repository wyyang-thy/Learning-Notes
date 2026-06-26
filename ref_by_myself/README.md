### 2026.06.26尝试自己组装雷氏七鳃鳗的精子参考基因组并且注释
#### 因为七鳃鳗在发育过程中存在大规模的体细胞 DNA 消除现象，鳃或血液等体细胞会丢失大量基因组片段，只有精子这样的生殖细胞才保留了最完整的真实基因组全貌。
#### 我的数据：
#### -rwxrwxr-x 1 vin vin 70G 6月   8 16:31 HIFI_14.bam
#### -rwxrwxr-x 1 vin vin 70G 6月   8 17:01 ont.fasta
#### 在这次试验中我准备使用tmux（Terminal Multiplexer）替代screen分离会话。下边记录一下它的基础用法：
```
# 新建一个会话，-s 参数用来指定会话的名字
tmux new -s lamprey_asm
# 进入会话后测试，检验服务器资源占用情况，ctrl c退出
top
# ctrl b两个健松开后按d退出会话，detach的意思，但是并不会杀死进程，在后台默认运行
# 查看哪些任务在后台跑
tmux ls
# 回到某一个进程，a 代表 attach（重连），-t 后面跟着你要重连的会话名字
tmux a -t lamprey_asm
# 彻底关闭对话有两种方法，第一种方法：在会话内输入exit回车或按ctrl d，第二种方法：
tmux kill-session -t lamprey_asm
```
#### 
