## 20260917以yang-asm1比对T2T判断breakpoint，并用petmar来做断点仲裁，以此例来深入了解基因组知识
### 基因组中的序列字符串本身永远是 5'→3' 书写的，不确定的是：这一串到底代表染色体的哪条链。所以当yang-asm1中的某一条contig作为query来minimap比对到T2T上时，以query的真实存储链向为+，如果能正好比对到reference上记录链向为+，如果需要query反向互补之后与reference才match则记录链向为-

### 根据基因组和注释从基因组中提取固定序列的CDS和蛋白质序列
```
# gffread中的这些选项没特殊的意义，是随便给的字母而已，格式：gffread -x <输出CDS文件> -g <基因组> <输入GTF>
# -w	转录本序列（外显子拼接，含 UTR）
# -x	CDS 序列（只有编码区的核酸）
# -y	蛋白序列（CDS 翻译成的氨基酸）
# -P：用 Perl 风格的正则表达式来解释搜索内容。提取的文件含这个基因的 gene、transcript、exon、CDS 等所有行。
grep -P 'gene_id "LrY_g027920"' ./yang-v2-annotation.gtf   > ./LrY_g027920.gtf
gffread -x LrY_g027920.CDS.fa -g yang-v2.polish.fa LrY_g027920.gtf
gffread -y LrY_g027920.protein.faa -g yang-v2.polish.fa LrY_g027920.gtf
```
