# 20260513记录对七鳃鳗数据进行cellranger分析，参考基因组使用petmar海七鳃鳗的参考基因组
## 先申请计算机点
```
srun -p cpu -n 64 --pty /bin/bash
```
## 因为之前的参考基因组没有外显子，所以我去ncbi上下载了新的GCF_010993605.1，传到了/lustre/home/acct-medcl/wyyang2025/workspace/cellranger_reference/pmar_ref_v2/GCF_010993605.1这个路径下，然后用这个参考基因组做了mkref，处理脚本如下
```
#SBATCH --partition=cpu
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=16
#SBATCH --mem=64G
#SBATCH --time=1-00:00:00
#SBATCH --output=mkref_%j.log
#SBATCH --error=mkref_%j.err

set -euo pipefail

CELLRANGER_HOME=/lustre/home/acct-medcl/wyyang2025/software/cellranger-10.0.0
export PATH="$CELLRANGER_HOME/bin:$PATH"
export TENX_IGNORE_DEPRECATED_OS=1

INPUT_DIR=/lustre/home/acct-medcl/wyyang2025/workspace/cellranger_reference/pmar_ref_v2/GCF_010993605.1
FA="$INPUT_DIR/GCF_010993605.1_kPetMar1.pri_genomic.fna"
GTF="$INPUT_DIR/genomic.gtf"

OUT_BASE=/lustre/home/acct-medcl/wyyang2025/workspace/cellranger_reference
REF_NAME=pmar_ref_v2_ref
WORK_DIR="$OUT_BASE/work_pmar_ref_v2"

mkdir -p "$WORK_DIR"
cd "$WORK_DIR"

# find transcripts that have no exon
awk '
match($0,/transcript_id "([^"]+)"/,a) && a[1] != "" {all[a[1]]=1}
$3=="exon"{match($0,/transcript_id "([^"]+)"/,a); if(a[1]!="") ex[a[1]]=1}
END{for(id in all) if(!(id in ex)) print id}
' "$GTF" > no_exon.txt

if [ -s no_exon.txt ]; then
  awk 'FNR==NR{bad[$1]=1; next}
       {match($0,/transcript_id "([^"]+)"/,a); if(!(a[1] in bad)) print}
  ' no_exon.txt "$GTF" > fixed.gtf
  GENES_GTF="$WORK_DIR/fixed.gtf"
else
  GENES_GTF="$GTF"
fi

if [ -e "$OUT_BASE/$REF_NAME" ]; then
  echo "Output already exists: $OUT_BASE/$REF_NAME" >&2
  exit 1
fi

cellranger mkref \
  --genome="$REF_NAME" \
  --fasta="$FA" \
  --genes="$GENES_GTF" \
  --nthreads=16 \
  --memgb=64
```
## 处理好的参考基因组在/lustre/home/acct-medcl/wyyang2025/workspace/cellranger_reference/work_pmar_ref_v2/pmar_ref_v2_ref这个路径下

## 写脚本
```
vim change_script.sh
```

## 输入
```
#!/bin/bash
#SBATCH --job-name=20260113-V1-5
#SBATCH --partition=cpu
#SBATCH --output=res_%j.log
#SBATCH --error=err_%j.log
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=16
#SBATCH --mem=100G
#SBATCH --time=7-00:00:00

set -euo pipefail

module purge

CELLRANGER_HOME=/lustre/home/acct-medcl/wyyang2025/software/cellranger-10.0.0
export PATH="$CELLRANGER_HOME/bin:$PATH"
export TENX_IGNORE_DEPRECATED_OS=1

REF=/lustre/home/acct-medcl/wyyang2025/workspace/cellranger_reference/work_pmar_ref_v2/pmar_ref_v2_ref
FASTQ_DIR=/lustre/home/acct-medcl/wyyang2025/workspace/Accuramed20260228/raw_data/20260113-V1-5
OUT_BASE=/lustre/home/acct-medcl/wyyang2025/workspace/Accuramed20260228/cellranger_by_myself
SAMPLE=20260113-V1-5

CORES=16
MEM=100

mkdir -p "$OUT_BASE"
cd "$OUT_BASE" || exit 1

echo "Cell Ranger: $(cellranger --version)"

cellranger count \
  --id="CR10_$SAMPLE" \
  --transcriptome="$REF" \
  --fastqs="$FASTQ_DIR" \
  --sample="$SAMPLE" \
  --include-introns=true \
  --create-bam=true \
  --nosecondary \
  --disable-ui \
  --localcores="$CORES" \
  --localmem="$MEM"
```
## 因为这次使用的试剂盒特别新，所以得要求Cellranger的版本很高，这里我使用的10.0.0，不然的话识别不到barcode
## 运行
```
sbatch change_script.sh
```

## 查看作业状态
```
squeue -j 57962096
```
## 输出，ST是PD意思是排队
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
          57962096       cpu 20260113 wyyang20 PD       0:00      1 (Priority)
## /lustre/home/acct-medcl/wyyang2025/workspace/cellranger_reference/pmar_ref现在这个路径下存放的是第一次mkref处理过的参考基因组，看样子这个已经没办法直接用了
## /lustre/home/acct-medcl/wyyang2025/workspace/cellranger_reference/pmar_ref_v2这个路径下存放的是原始的新下载的参考基因组以及mkref的脚本
## /lustre/home/acct-medcl/wyyang2025/workspace/cellranger_reference/work_pmar_ref_v2这个路径是这一次做的新的参考基因组的工作路径
## /lustre/home/acct-medcl/wyyang2025/workspace/cellranger_reference/work_pmar_ref_v2/pmar_ref_v2_ref这个路径下存放的是这一次处理好的参考基因组，这一次是可以使用的
