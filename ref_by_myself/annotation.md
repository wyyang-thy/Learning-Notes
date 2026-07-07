# 基因组的注释学习文档
## 首先需要明确的是，三代也可以测RNA转录组也可以测基因组，区别在于上机的样品不同。直接从七鳃鳗细胞里提取染色体 DNA，打断后上机器这是全基因组测序WGS，主要用于基因组的组装。如果提取了 RNA，逆转录成 cDNA，然后交给 PacBio 机器测序，这在业内叫做 Iso-Seq（全长转录组测序），用 ONT 机器测，叫 ONT-cDNA 测序。这不仅能测，而且效果比二代更好，因为它能直接把一条完整的转录本（mRNA）从头到尾读下来。
## 注释之前需要构建基因模型，有三种策略：
### 1、从头注释(de novo prediction)：在没有已知模板、参考序列或历史经验数据作为比对基础的情况下，通过已有的概率模型来预测基因结构，在预测剪切位点和UTR区准确性较低。用 Augustus 或 GeneMark
#### 注意：从头预测工具（如 Augustus、SNAP）的核心是统计学模型（通常是隐马尔可夫模型 HMM）。这些模型不知道要注释的这个新物种的基因长什么样。不同物种的基因特征差异巨大（比如 GC 含量、密码子偏好性、内含子平均长度）。如果直接拿人类的基因特征模型去预测植物的基因组，结果会惨不忍睹。因此，在正式让工具去全基因组里找基因之前，必须先找一批该物种“最标准、最完美”的真实基因，当作“课本”喂给这些软件，让它们学习（训练），然后才能让它们去预测。用来训练模型的转录组数据必须是要注释的这个物种的，因为它需要学习这个物种的特征。

### 2、同源预测(homology-based prediction)：利用已知物种的基因、转录本或蛋白质序列作为线索或模板，通过序列比对的方式，在新基因组中寻找相似的序列，从而确定新基因的位置和结构。有一些基因蛋白在相近物种间的保守型高，所以可以使用已有的高质量近缘物种注释信息通过序列联配的方式确定外显子和内含子边界和剪切位点。用 Exonerate 或 miniprot

### 3、基于转录组预测(transcriptome-based prediction)：不管是bulk测序还是单细胞RNA测序都是从细胞中得到已经表达的RNA，把这些测序得到的Reads比对到拼装好的基因组上就能准确知道哪些是外显子，能够比对上的肯定就是外显子也就是编码区，并且能够发现“可变剪接”（Alternative Splicing，即同一个基因拼出不同版本的 mRNA。同一基因的前体mRNA通过不同剪切方式生成多种成熟mRNA，从而产生不同蛋白质的过程）。通过物种的RNA-seq数据辅助注释，能够较为准确的确定剪切位点（指的是内含子和外显子的分界点，5'端总是GT，3'端总是AG）和外显子区域。用 StringTie 配合 HISAT2

### 每一种方法都有自己的优缺点，所以最后需要用EvidenceModeler(EVM)和GLEAN工具进行整合，合并成完整的基因结构。基于可靠的基因结构，后续可才是功能注释，蛋白功能域注释，基因本体论注释，通路注释等。

## 基因组注释前准备
### 雷氏七鳃鳗拉丁名：Lethenteron reissneri
### 5个同源物种（需要有注释的基因/蛋白序列，保证高组装和注释质量）：
#### 1、北极七鳃鳗 / 日本七鳃鳗 (Arctic lamprey)：拉丁学名: Lethenteron camtschaticum (曾用名: Lethenteron japonicum)。NCBI 组装编号: GCF_018977245.1 (版本: IMCB_Ljap_1.0)。与目标物种雷氏七鳃鳗（Lethenteron reissneri）亲缘关系最近的（同属于叉牙七鳃鳗属 Lethenteron）
#### 2、海七鳃鳗 (Sea lamprey)。拉丁学名: Petromyzon marinus。NCBI 组装编号: GCF_010993605.1 (版本: kPetMar1.pri)。这是整个无颌类和七鳃鳗研究领域的“模式生物”。
#### 3、欧洲河七鳃鳗 (European river lamprey).拉丁学名: Lampetra fluviatilis。NCBI 组装编号: GCF_964198595.1 (版本: kcLamFluv1.1)。数据比较新。
#### 4、欧洲溪七鳃鳗 (European brook lamprey).拉丁学名: Lampetra planeri。NCBI 组装编号: GCF_965212315.1 (版本: kcLamPlan1.2.h2)。数据比较新。
#### 5、太平洋七鳃鳗 (Pacific lamprey)。拉丁学名: Entosphenus tridentatus。NCBI 组装编号: GCA_014621495.2 (版本: ETRf_v1)。
### 转录组数据: 雷氏七鳃鳗的RNAseq和lsoseq注释（用于结构注释中的转录辅助注释）：
#### Bulk RNA-seq 数据：项目编号：PRJNA558325。研究人员提取了雷氏七鳃鳗的 10 个不同器官（心脏、鳃、精巢、脑、肝脏、口腔腺、肾脏、肠道、神经上体和肌肉）进行了高质量的 RNA 测序。SRA 编号 (Run ID)： 从 SRR9964076 到 SRR9964085。这 10 个文件下载下来一起比对，能提升基因组外显子边界预测的准确性。
#### 长读长全长转录组 (Iso-Seq) 数据：：在 NCBI SRA 数据库的主页（ncbi.nlm.nih.gov/sra）搜索框中输入：Lethenteron reissneri Iso-seq 或者 Lethenteron reissneri PacBio 来获取最新的 SRA 编号。

## 基因组注释的分析内容
### 重复注释
#### 重复序列广泛存在于真核生物基因组中，这些重复序列或集中成簇，或分散在基因之间。根据分布把重复序列分为散在重复序列和串联重复序列。重复序列根据序列特征分为2类：串联重复（Tandem repeats，如微卫星序列(150bp,13bp)，小卫星序列(20kb,25bp)，卫星序列(<5bp,>200bp)）和散布重复（Dispersed repeats，如DNA转座子，长末端重复序列LTR(1.5kbp-10kbp)，长散在重复序列LINE(>1000bp)，短散在重复序SINE(about300bp)）(每个软件都有很多参数,可-help/-h自行查看,参数的选择最好是参考已发表的文献)
##### RepeatMasker:基于Repbase(dna)/自建elibrary查询重复序列
```
RepeatMasker -nolow -no_is -norna -parallel 2 -lib RepeatMasker.lib genome.fa
#-nohow:屏蔽低复杂简单重复; -no_is:跳过细菌插入元件检查; -norna:不掩盖小RNA(伪)基因;
#-parallel 并行使用的处理器数,可提升分析速度
```
##### RepeatProteinMask:基于 Repbase(pep)查询重复序列
```
RepeatProteinMask -noLowSimple -pvalue 0.0001 genome.fa
#noLowSimple:关闭低复杂度和简单重复的屏蔽/注释; -pvalue:接受匹配的阈值
#注意点: genome.fa的D不能长于18个字符
```
##### TRF:元件的结构特征等来识别重复序列
```
trf genome.fa 2 7 7 80 10 50 2000 -d -h
```
##### LTR-FINDER:基于重复序列特征
```
Itr_finder -W 2 -C -s tRNAs.fa genome.fa
#-w 2 输出格式,2-table;  -C:检测中心粒,删除高重复区域
```
##### repeatmodeler:基于自身序列比对
```
BuildDatabase -name mydb genome.fa
RepeatModeler -database mydb -pa 6 >run.out
#-name:创建 database的名称;
#-pa:共享内存处理器的数量程序,可提升分析速度
```
### 结构注释
#### 结构注释:注释可以产生具有生物学功能的蛋白的基因。一般包括启动子，转录起始，5’UTR，起始密码子，外显子，内含子，终止密码子，3’UTR，poly-A等结构。以下是从大到小的包含关系
##### De novo预测(屏蔽重复序列)
###### Augustus(真核)
```
augustus --species=XXX --AUGUSTUS CONFIG PATH= config --uniqueGeneld=true --nolnFrameStop=true--gff3=on --strand=both genome.mask.fa> genome.mask.fa.out
# --uniqueGeneld=true:gene:命名 aseqname.gn;
# --nolnFrameStop=true:不带有终止密码子的转录本;
# --gff3=on:输出格式gff3
```
###### GlimmerHMM(真核,预测的基因数目较多长度较短,一般用于植物)
```
glimmerhmm.genome.mask.fa -d XXX- f -g genome.mask.fa.gff

# -d 库de路径;
# -f:不要partial gene predictions;
# -g输出格式gff
```
###### Genscan(真核,其预测的内含子较大,一般用于动物)
```
genscan Humanlso.smat genome.mask.fa > genome.mask.fa.genscan
# Humanlsc.smat:参数文件,软件自带
```
###### 其他软件
```
SNAP. GenelD GenemarkS
denovo的软件很多,两个软件就可以了,太多软件会增加较多的假阳性,一般在
Augustus, GlimmerHMM, Genscan中选择即可
```
##### Homolog注释
###### 利用近缘物种已知基因进行序列比对,找到同源序列。然后在同源序列的基础上,根据基因信号如剪切信号、基因起始和终止密码子对基因结构进行预测。相对于从头预测的“大海捞针”,同源预测相当于先用一块磁铁在基因组大海中缩小了可能区域,然后从可能区域中鉴定基因结构。利用TBlastn将同源物种的蛋白比对回基因组,得到候选区域。利用 EXonerate/ Genewise进行精确的蛋白-核酸比对,以得到剪接位点。Exonerate解决了 GeneWisez存在的很多问题,并且速度快了1000倍,默认选择EXonerate分析

##### RNA-seq辅助注释.tophat比对————>cufflink转录本————>TransDecoder。将RNAseq数据进行tophat比对;比对后的结果文件利用cufflink构建转录本使用TransDecoder在构建的转录本上预测Open Reading Frame(ORF)。

##### Iso seq 辅助注释.CD-HIT————>gmap比对————>TransDecoder。将物种的三代全长转录本用CD-HIT进行去冗余;将去冗余后的序列使用gmap比对回基因组得到转录本位置;使用TransDecoder在构建的转录本上预测 Open Reading Frame(ORF).

### MAKERE整合
#### 在基因组注释上, MAKER算是一个很强大的分析流程,主要是进行 Denovo注释， Homolog注释,转录辅助注释三者的整合,保证最终注释基因集的可靠性
```
maker maker_exe.ctl maker_opts.ctl maker_bopts.ctl
#maker exe.ct:执行程序的路径
#maker_ boots.ctl: BLAST7和 Exonerate的过滤参数
#maker opts.ctl:其他信息,例如输入基因组文件,主要调整输入文件等( genome= ;est= ;protein= ;pred_gff= ;)
```

### nCRNA注释
```
rRNA(核糖体RNA)
·与蛋白质结合形成核糖体,其功能是作为mn的支架,提供mRNA翻译成蛋白质的场所。
tRNA(转运RNA)
·携带氨基酸进入核糖体,使之在mRNA指导下合成蛋白质。
miRNA(miRNA)
·将mRNA降解或抑制其翻译,具有沉默基因的功能。
SnRNA(小核RNA)
·主要参与RNA前体的加工过程,是RNA剪切体的主要成分。
```
#### miRNA与snRNA注释
```
采用Rfam和INFERNAL进行二级结构检测。
ftp://ftp.sanger.ac.uk/pub/databases/Rfam
blastn+cmsearch (INFERNAL程序)
```
#### rRNA注释
```
由于rRNA的结构保守程度非常高，因此采用与已有的全长rRNA进行blastn比对而获得。
blastn
tRNA注释
结构特点:三叶草型二级结构。
预测方法:针对二级结构进行检测。使用tRNAscan-SE
```

### 功能注释
#### 功能注释:基因功能的注释依赖于上一步的基因结构预测，根据预测结果从基因组上提取翻译后的蛋白序列和主流的数据库进行blastp比对，完成功能注释。

##### 常用数据库一共有以下几种:NR，KEGG, Uniprot (Swiss-Prot, TrEMBL)，InterPro,Go
```
KEGG
生物学通路数据库(Gene,Pathway,Ligand).
KEGG: Kyoto Encyclopedia of Genes and Genomes
blastp
SWISS-PROT和TrEMBL
UniProt (Universal Protein Resource)蛋白质序列数据库PIR、SWISS-PROT和TrEMBL统一起来，建立了一个蛋白质数据库。
UniProt
blastp
Interpro
蛋白家族(protein families)、功能保守区域(domains)和功能位点(funtional sites)的数据库.
InterPro
InterProScan
GO
基因功能注释数据库(GeneOntology)
三个层面Cellular Component、 Biological Process、 Molecular Function.
Gene Ontology Resource
InterProScan
```
### 基因组评估
#### BUSCO评估
##### BUSCO是一款使用python语言编写的对转录组和基因组组装质量进行评估的软件。在相近的物种之间总有一些保守的序列，而BUSCO就是使用这些保守序列与组装的结果进行比对，鉴定组装的结果是否包含这些序列，包含单条、多条还是部分或者不包含等等情况来给出结果。BUSCO软件根据OrthoDB数据库，构建了几个大的进化分支的单拷贝基因集。将其与该基因集进行比较，根据比对上的比例、完整性，来评价准确性和完整性。
