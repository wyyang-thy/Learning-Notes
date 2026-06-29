### 2026.06.26尝试自己组装雷氏七鳃鳗的精子参考基因组并且注释
#### 因为七鳃鳗在发育过程中存在大规模的体细胞 DNA 消除现象，鳃或血液等体细胞会丢失大量基因组片段，只有精子这样的生殖细胞才保留了最完整的真实基因组全貌。
#### 我的数据：
#### -rwxrwxr-x 1 vin vin 70G 6月   8 16:31 HIFI_14.bam，这个数据的特点是很精确，准确率很高，用于精确构建contig。Reads → Contigs → Scaffolds → （Chromosomes）
#### -rwxrwxr-x 1 vin vin 70G 6月   8 17:01 ont.fasta，特点是超长读长，可以跨过大片重复区域、转座子、中心粒等复杂区域
### 说明：
#### Reads 是测序仪直接输出的原始序列片段，就是 HiFi 和 ONT 数据。
##### Contigs（连续序列）是组装软件把有重叠的 reads 拼接在一起形成的连续序列，内部没有任何缺口，是"确定知道"的序列。但 contig 和 contig 之间的顺序和距离往往还不清楚。
##### Scaffolds 是把多个 contig 按照推断出的顺序和方向连接起来的结果，contig 之间不明确的部分用 N 填充（表示"这里有序列，但不知道是什么"）。建 scaffold 通常需要额外的信息，比如 Hi-C 数据、光学图谱，或者你手里 ONT 的超长读长。
##### 染色体级别组装是最终目标，理想情况下一条染色体对应一条几乎没有 N 的完整序列。
```
hifiasm 组装完成（.gfa 文件）
         ↓
① GFA → FASTA 转换（得到 contig 序列）
         ↓
② 组装质量评估（N50、BUSCO、QV 值）
         ↓
③ 后续 scaffold（如果有 Hi-C）或直接进入下游分析
```

### HiFi 做骨架：利用高精度 read 先组装出高质量的 contig，内部碱基误差极少
### ONT 补全结构：用超长 read 连接 contig、填补 gap，解决 HiFi 读长不足以跨越的大型重复区

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
```
# 查看多少个核心，实验室服务器192个核心
nproc
# 查看内存
free -h
```
#### 服务器上没有hifiasm
```
git clone https://github.com/chhylp123/hifiasm.git
# 进入目录并极速编译（-j 32 表示调用 32 个线程加速编译）：
cd hifiasm
make -j 32
# 验证是否安装成功：
./hifiasm --version
```
### 第一步：调用 hifiasm 进行 HiFi + ONT 超长读长的混合组装。hifiasm 做的事情大致是：读入 HiFi reads，构建重叠图（overlap graph）；利用 ONT 超长读长辅助解析重复区、桥接 contig；输出一系列 .gfa 格式的图文件
#### 写脚本
```
vim run_hybrid_assembly.sh
```
#### 脚本内容
```
#!/bin/bash
set -euo pipefail

# --- 变量设置 ---
WORK_DIR="/tmpdata/YangWenyan/lamprey_assembly"
RAW_DATA_DIR="/home/YangWenyan/workspace/T2T/T2T_rawdata"
HIFIASM="/home/YangWenyan/workspace/ref_by_myself/hifiasm/hifiasm"
THREADS=120
PREFIX="lamprey_hybrid"
GFA="${PREFIX}.bp.p_ctg.gfa"
FASTA="${PREFIX}.p_ctg.fasta"
LOG="${PREFIX}_run.log"

mkdir -p "${WORK_DIR}"
cd "${WORK_DIR}"

echo "========== Hybrid Assembly Started at $(date) =========="

# Step 1: BAM 转 FASTQ
echo "[Step 1] Converting BAM to FASTQ..."
if [ ! -f "HIFI_14.fastq" ]; then
    samtools fastq -@ "${THREADS}" "${RAW_DATA_DIR}/HIFI_14.bam" > HIFI_14.fastq \
        || { echo "ERROR: samtools fastq failed" >&2; exit 1; }
    echo "[Step 1] Conversion complete: $(wc -l < HIFI_14.fastq) lines written."
else
    echo "[Step 1] HIFI_14.fastq already exists, skipping."
fi

# Step 2: hifiasm 混合组装
echo "[Step 2] Running hifiasm hybrid assembly..."
"${HIFIASM}" -o "${PREFIX}" -t "${THREADS}" \
    --ul "${RAW_DATA_DIR}/ont.fasta" \
    HIFI_14.fastq \
    2>&1 | tee "${LOG}"

if [ "${PIPESTATUS[0]}" -ne 0 ]; then
    echo "ERROR: hifiasm failed. Check ${LOG} for details." >&2
    exit 1
fi
echo "[Step 2] hifiasm finished at $(date)"

# Step 3: GFA 转 FASTA
echo "[Step 3] Extracting primary contigs from GFA..."
if [ ! -f "${GFA}" ]; then
    echo "ERROR: GFA file not found: ${GFA}" >&2
    echo "Available GFA files:" >&2
    ls "${PREFIX}"*.gfa 2>/dev/null || echo "  (none found)" >&2
    exit 1
fi

awk '/^S/{print ">"$2"\n"$3}' "${GFA}" > "${FASTA}"
echo "[Step 3] FASTA written: ${FASTA}"

# Step 4: 组装统计
echo "[Step 4] Calculating assembly statistics..."
seqkit stats -a "${FASTA}" | tee "${PREFIX}_seqkit_stats.txt"

echo "========== Hybrid Assembly Finished at $(date) =========="
```
#### 脚本精简版
```
#!/bin/bash

# 变量设置
WORK_DIR="/tmpdata/YangWenyan/lamprey_assembly"
RAW_DATA_DIR="/home/YangWenyan/workspace/T2T/T2T_rawdata"
HIFIASM="/home/YangWenyan/workspace/ref_by_myself/hifiasm/hifiasm"
THREADS=120
PREFIX="lamprey_hybrid"

# 进入专属数据盘工作目录
mkdir -p ${WORK_DIR}
cd ${WORK_DIR}

# 第一步：BAM 转 FASTQ
samtools fastq -@ ${THREADS} ${RAW_DATA_DIR}/HIFI_14.bam > HIFI_14.fastq

# 第二步：hifiasm 混合组装
${HIFIASM} -o ${PREFIX} -t ${THREADS} --ul ${RAW_DATA_DIR}/ont.fasta HIFI_14.fastq

# 第三步：GFA 转 FASTA
awk '/^S/{print ">"$2"\n"$3}' ${PREFIX}.bp.p_ctg.gfa > ${PREFIX}.p_ctg.fasta

# 第四步：组装统计
seqkit stats -a ${PREFIX}.p_ctg.fasta > ${PREFIX}_seqkit_stats.txt
```
#### 加运行
```
chmod +x run_hybrid_assembly.sh
```
#### 运行脚本并实时保存日志（tee 命令可以在屏幕上显示输出的同时，把日志写进 assembly_log.txt 里备查）：
```
./run_hybrid_assembly.sh
```
